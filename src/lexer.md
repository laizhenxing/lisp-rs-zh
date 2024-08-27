# 词法分析器

## 源码

[lexer.rs](https://github.com/vishpat/lisp-rs/blob/0.0.1/src/lexer.rs)

## 代码演练

<b>Lexer</b> 是解释器的一个组件，它获取程序文本并将其转换为称为标记的原子单元流。每个令牌都有一个类型，甚至可能有一个与之关联的值。就我们的解释器而言，有四种类型的令牌。这四个标记可以使用 Rust 枚举来表示，如下所示。

```rust
pub enum Token {
    Integer(i64),
    Symbol(String),
    LParen,
    RParen,
}
```

- `Integer`: 有符号的 64 位整数
- `Symbol`: 除整数或括号之外的任何字符组
- `LParen`: 左括号
- `RParen`: 右括号

实现词法分析器的第一步是用括号前后的额外空格替换括号。例如，

```lisp
(define sqr (* x x))
```

将被转换为

```lisp
( define sqr (* x x ) )
```

通过这个简单的技巧，标记化的过程涉及到用空格分割 Lisp 程序。

```rust
let program2 = program.replace("(", " ( ").replace(")", " ) ");
let words = program2.split_whitespace();
```

一旦获得了程序的单词，就可以使用 Rust 的模式匹配将它们转换为标记，如下所示

```rust
let mut tokens: Vec<Token> = Vec::new();
for word in words {
    match word {
        "(" => tokens.push(Token::LParen),
        ")" => tokens.push(Token::RParen),
        _ => {
             let i = word.parse::<i64>();
                if i.is_ok() {
                  tokens.push(Token::Integer(
                  	i.unwrap()));
                } else {
                  tokens.push(Token::Symbol(
                  	word.to_string()));
                }        
            }
    }
}
```

此时，我们就有了整个 `Lisp` 程序的标记向量。请注意，`Rust` 中的 `vector` 是一个堆栈，因此标记以相反的顺序存储在向量中，如下例所示。

![tokens](https://vishpat.github.io/lisp-rs/images/token_stack.png)

## 测试

词法分析器代码可以进行单元测试，如下所示

```rust
let tokens = tokenize("(+ 1 2)");
assert_eq!(
    tokens,
    vec![
        Token::LParen,
        Token::Symbol("+".to_string()),
        Token::Integer(1),
        Token::Integer(2),
        Token::RParen,
    ]
);
```

为了巩固您对 `Lexing` 流程的理解，请完成 `lexer.rs` 中的其余测试。