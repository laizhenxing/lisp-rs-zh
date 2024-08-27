# 解释器

## 源码

[parser.rs](https://github.com/vishpat/lisp-rs/blob/0.0.1/src/parser.rs)

## 代码演练

解析器的工作是获取 `tokens vector` 并将其转换为递归列表结构（在[简介](https://vishpat.github.io/lisp-rs/introduction.html)中提到）。这个递归列表结构是 Lisp 程序的内存表示。

## 对象模型

在深入了解如何创建递归列表结构的细节之前，我们需要首先定义构成列表元素的对象。这是在对象Rust 模块中完成的，枚举定义如下

```rust
pub enum Object {
    Void,
    Integer(i64),
    Bool(bool),
    Symbol(String),
    Lambda(Vec<String>, Vec<Object>),
    List(Vec<Object>),
}
```

- `Void`: 空对象
- `Integer`: 有符号 64 位整数
- `Bool`: 布尔值
- `Symbol`: Lisp 符号，类似于 Symbol 标记
- `Lambda`: 函数对象，第一个向量表示参数标签，第二个向量表示函数体。该对象在解析期间不使用，但将在解释器的评估阶段使用。
- `List`: 列表对象

## 解释器

Lisp 方言的解析器是一个简单的递归函数。它采用由词法分析器生成的 `tokens vector`（反向），并生成代表整个 Lisp 程序的单个列表对象。解析器的核心逻辑是由递归的 `parse_list` 函数实现的，如下所示。它需要一个 `tokens vector` ，`vector` 的第一个标记是左括号，指示列表的开头。然后该函数继续一次处理列表中的元素。如果它遇到原子 `token`（例如整数或符号），它会创建相应的原子对象并将它们添加到列表中。如果遇到另一个左括号，它将使用剩余的标记进行递归。最后，当函数遇到右括号时，返回列表对象。

```rust
let token = tokens.pop();
if token != Some(Token::LParen) {
    return Err(ParseError {
        err: format!("Expected LParen, found {:?}", 
                     token),
    });
}
   
let mut list: Vec<Object> = Vec::new(); 
while !tokens.is_empty() {
    let token = tokens.pop();
    if token == None {
        return Err(ParseError {
            err: format!("Insufficient tokens"),
        });
    }
    let t = token.unwrap();
    match t {
        Token::Integer(n) => 
            list.push(Object::Integer(n)),
        Token::Symbol(s) => 
            list.push(Object::Symbol(s)),
        Token::LParen => {
            tokens.push(Token::LParen);
            let sub_list = parse_list(tokens)?;
            list.push(sub_list);
        }
        Token::RParen => {
            return Ok(Object::List(list));
        }
    }
}
```

## 测试

上述解析代码可以进行单元测试如下

```rust
let list = parse("(+ 1 2)").unwrap();
assert_eq!(
    list,
    Object::List(vec![
        Object::Symbol("+".to_string()),
        Object::Integer(1),
        Object::Integer(2),
    ])
);
```

为了巩固您对解析过程的理解，请完成`parser.rs`中的其余测试