# REPL

## 源码

[main.rs](https://github.com/vishpat/lisp-rs/blob/0.0.1/src/main.rs)

## 代码演练

`REPL` 是解释器实现中最简单的部分。它使用 `linefeed crate` 来实现该功能。

`REPL` 首先创建一个env实例来保存解释器的全局变量。

```rust
let mut env = Rc::new(RefCell::new(Env::new()));
```

`REPL` 是一个简单的 `while` 循环，它将一行作为用户的输入，对其进行计算，然后打印出计算后的对象。如果用户输入 `exit` 关键字，`REPL` 将终止。

```rust
while let ReadResult::Input(input) = 
  			  reader.read_line().unwrap() {
    
    if input.eq("exit") {
        break;
    }
    let val = eval::eval(input.as_ref(), &mut env)?;
    match val {
        Object::Void => {}
        Object::Integer(n) => println!("{}", n),
        Object::Bool(b) => println!("{}", b),
        Object::Symbol(s) => println!("{}", s),
        Object::Lambda(params, body) => {
            println!("Lambda(");
            for param in params {
                println!("{} ", param);
            }
            println!(")");
            for expr in body {
                println!(" {}", expr);
            }
        }
        _ => println!("{}", val),
    }
}
```