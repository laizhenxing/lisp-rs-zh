# 评估器

现在是该项目最令人兴奋的部分。评估是产生 Lisp 程序结果的最后一步。在较高级别上，评估器函数递归地遍历解析器创建的基于列表的结构，并（递归地）评估每个原子对象和列表，组合这些中间值，并产生最终结果。

## 源码

[eval.rs](https://github.com/vishpat/lisp-rs/blob/0.0.1/src/eval.rs)

## 代码演练

评估器是使用递归`eval_obj`函数实现的。 `eval_obj`函数将表示程序的 `List` 对象和全局环境变量（一个简单的哈希图）作为输入。然后，该函数开始通过迭代该列表的每个元素来处理表示程序的 `List` 对象

```rust
fn eval_obj(obj: &Object, env: &mut Rc<RefCell<Env>>) 
    -> Result<Object, String> 
{
    match obj {
        Object::Void => Ok(Object::Void),
        Object::Lambda(_params, _body) => Ok(Object::Void),
        Object::Bool(_) => Ok(obj.clone()),
        Object::Integer(n) => Ok(Object::Integer(*n)),
        Object::Symbol(s) => eval_symbol(s, env),
        Object::List(list) => eval_list(list, env),
    }
}
```

对于原子对象（例如整数和布尔值），评估器仅返回对象的副本。对于 `Void` 和 `Lambda`（函数对象），它返回 `Void` 对象。现在，我们将介绍实现评估器大部分功能的 `eval_symbol` 和 `eval_list` 函数。

### eval_symbol

在了解 `eval_symbol` 函数之前，了解 `Lisp` 解释器如何实现变量的设计非常重要。

变量只是分配给值的字符串标签，它们是使用 `Define` 关键字创建的。请注意，可以为变量分配原子值，例如整数或布尔值，也可以为它分配函数对象

```lisp
(
    (define x 1)
    (define sqr (lambda (r) (* r r)))
)
```

上面的 `Lisp` 代码定义（或创建）两个名为 `x` 和 `sqr` 的变量，分别表示整数和函数对象。此外，这些变量的范围位于定义它们的列表对象内。这是通过将变量名称（字符串）到对象的映射存储在名为 `Env` 的类似映射的数据结构中来实现的，如下所示。

```rust
// env.rs
pub struct Env {
    parent: Option<Rc<RefCell<Env>>>,
    vars: HashMap<String, Object>,
}
```

解释器在程序开始时创建一个 `Env` 实例来存储全局变量定义。此外，对于每个函数调用，解释器都会创建一个新的 `env` 实例，并使用新实例来评估函数调用。这个新的 `env` 实例包含函数参数以及指向调用该函数的父 `env` 实例的后向指针，如下例所示

```lisp
(
	(define m 10)
	(define n 12)
	(define K 100)
	
	(define func1 (lambda (x) (+ x K)))
	(define func2 (lambda (x) (- x K)))
	
	(func1 m)
	(func2 n)
)
```

![env](https://vishpat.github.io/lisp-rs/images/env.png)

当我们浏览代码时，这个概念将变得更加清晰。

`eval_symbol` 的工作是查找绑定到该符号的对象。这是通过递归查找传递的环境变量或其任何父环境直到程序的根环境来完成的。

```rust
let val = env.borrow_mut().get(s);
if val.is_none() {
    return Err(format!("Unbound symbol: {}", s));
}
Ok(val.unwrap().clone())
```

### eval_list

`eval_list` 函数是评估器的核心，其实现如下所示。

```rust
let head = &list[0];
match head {
    Object::Symbol(s) => match s.as_str() {
        "+" | "-" | "*" | "/" | "<" | ">" | "=" | "!=" => {
            return eval_binary_op(&list, env);
        }
        "if" => eval_if(&list, env),
        "define" => eval_define(&list, env),
        "lambda" => eval_function_definition(&list),
        _ => eval_function_call(&s, &list, env),
    },
    _ => {
        let mut new_list = Vec::new();
        for obj in list {
            let result = eval_obj(obj, env)?;
            match result {
                Object::Void => {}
                _ => new_list.push(result),
            }
        }
        Ok(Object::List(new_list))
    }
}
```

此函数查看列表的头部，如果头部与符号对象不匹配，它将迭代列表的所有元素，递归地评估每个元素并返回一个包含评估对象值的新列表。

### 变量定义

如果 `eval_list` 函数中的列表头与 `define` 关键字匹配，例如

```lisp
(define sqr (lambda (x) (* x x)))
```

`eval_define` 函数对列表的第三个参数调用 `eval_obj`，并将计算的对象值分配给列表中第二个参数定义的符号。然后，该符号及其对象值将存储在当前 `env` 中。

```rust
let sym = match &list[1] {
    Object::Symbol(s) => s.clone(),
    _ => return Err(format!("Invalid define")),
};
let val = eval_obj(&list[2], env)?;
env.borrow_mut().set(&sym, val);
```

在上面的示例中，符号 `sqr` 和表示 `lambda` 的函数对象将存储在当前 `env` 中。一旦以这种方式定义了函数 `sqr`，后面的任何代码都可以通过在 `env` 中查找符号 `sqr` 来访问相应的函数对象。

### 二元运算

如果 `eval_list` 函数中的列表头与二元运算符匹配，则根据二元运算符的类型评估列表，例如

```lisp
(+ x y)
```

`eval_binary_op` 函数对列表的第二个和第三个元素调用 `eval_obj`，并对评估值执行二进制求和运算。

### if 语句

例如，如果 `eval_list` 函数中的列表头与 `if` 关键字匹配

```lisp
(if (> x y) x y)
```

`eval_if` 函数对列表的第二个元素调用 `eval_obj`，并根据评估值是 `true` 还是 `false`，对列表的第三个或第四个元素调用 `eval_obj` 并返回值

```rust
let cond_obj = eval_obj(&list[1], env)?;
let cond = match cond_obj {
    Object::Bool(b) => b,
    _ => return Err(format!("Condition must be a boolean")),
};

if cond == true {
    return eval_obj(&list[2], env);
} else {
    return eval_obj(&list[3], env);
}
```

### Lambda

如前所述， `lambda` （或函数）对象由两个 `vector` 组成

```rust
Lambda(Vec<String>, Vec<Object>>)
```

如果 `eval_list` 函数中的列表头与 `lambda` 关键字匹配，例如

```lisp
(lambda (x) (* x x))
```

`eval_function_definition` 函数将列表的第二个元素计算为参数名称 `vector`。

```rust
let params = match &list[1] {
    Object::List(list) => {
        let mut params = Vec::new();
        for param in list {
            match param {
                Object::Symbol(s) => params.push(s.clone()),
                _ => return Err(format!("Invalid lambda parameter")),
            }
        }
        params
    }
    _ => return Err(format!("Invalid lambda")),
};
```

列表的第三个元素被简单地克隆为函数体。

```rust
let body = match &list[2] {
    Object::List(list) => list.clone(),
    _ => return Err(format!("Invalid lambda")),
};
```

```rust
Ok(Object::Lambda(params, body))
```

评估的参数和主体 `vector` 作为 `lambda` 对象返回

### 函数调用

如果列表的头部是一个 `Symbol` 对象，并且它与任何上述关键字或二元运算符都不匹配，则解释器假定该 `Symbol` 对象映射到 `Lambda`（函数对象）。 `Lisp` 中函数调用的例子如下

```lisp
(find_max a b c)
```

为了评估这个列表，调用 `eval_function_call` 函数。该函数首先使用函数名称 `find_max` 来查找函数对象，在此例中为 `find_max`。

```rust
let lamdba = env.borrow_mut().get(s);
if lamdba.is_none() {
    return Err(format!("Unbound symbol: {}", s));
}
```

如果找到该函数对象，则会创建一个新的 `env` 对象。这个新的 `env` 对象有一个指向父 `env` 对象的指针。这是获取函数范围内未定义的变量值所必需的。

```rust
let mut new_env = Rc::new(
    			  RefCell::new(
    			  Env::extend(env.clone())));
```

评估函数调用的下一步需要准备函数参数。这是通过迭代列表的其余部分并评估每个参数来完成的。然后，参数名称和评估的对象将设置在新的 `env` 对象中。

```rust
for (i, param) in params.iter().enumerate() {
    let val = eval_obj(&list[i + 1], env)?;
    new_env.borrow_mut().set(param, val);
}
```

最后，通过传递 `new_env` 来评估函数体，其中包含函数的参数

```rust
let new_body = body.clone();
return eval_obj(&Object::List(new_body), &mut new_env);
```
