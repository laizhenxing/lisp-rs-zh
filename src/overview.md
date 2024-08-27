# 概览
[Lisp-rs](https://github.com/vishpat/lisp-rs) 项目在 Rust 中为 [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) 的一个小子集(一种 Lisp 方言)实现了一个解释器。本文的主要目的是确保读者了解解释器是如何实现的。

这个项目的灵感来自 Peter Norvig 的文章[(How to Write a (Lisp) Interpreter (In Python))](http://www.norvig.com/lispy.html)和书籍 [WritingAn Interpreter In Go](https://interpreterbook.com/)。本文档作为实现解释器的[代码](https://github.com/vishpat/lisp-rs/tree/0.0.1)的注释。Rust 丰富的编程构造，例如 enum、模式匹配和错误处理，使得实现这个基本解释器变得非常容易和有趣。

## 先决条件

为了充分利用该项目，用户需要了解以下计算机科学概念。

- [列表](https://en.wikipedia.org/wiki/List_(abstract_data_type))
- [递归](https://en.wikipedia.org/wiki/Recursion_(computer_science))

Rust 是一种不平凡的语言，但是，要实现 Lisp 解释器，读者需要对该语言有一定的经验。了解以下四个概念应该足以让用户理解整个项目。
- [枚举和模式匹配](https://doc.rust-lang.org/book/ch06-00-enums.html)
- [智能指针](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html)
- [错误处理](https://doc.rust-lang.org/book/ch09-00-error-handling.html)


## Lisp 语言
为了保持解释器简单且其实现易于理解，其支持的功能数量已被有意限制。以下是解释器支持的数据类型和语句。

### 数据类型
- integer
- boolean

### 声明
- 变量定义和赋值
- if-else
- 使用 lammbda 定义函数
- 函数调用

### 关键字
- define
- if-else
- lambda

### 示例
以下是您可以使用解释器运行的一些示例程序。

```lisp
    (
        (define factorial (lambda (n) (if (< n 1) 1 (* n (factorial (- n 1))))))
        (factorial 5)
    )
```

```lisp
    (
        (define pix 314)
        (define r 10)
        (define sqr (lambda (r) (* r r)))
        (define area (lambda (r) (* pix (sqr r))))
        (area r)
    )
```

## 解释器
解释器将从头开始实现，不需要[nom](https://docs.rs/nom/latest/nom/)或[pest](https://pest.rs/)等任何工具的帮助。解释器的实现分为四个部分

- [词法分析](https://vishpat.github.io/lisp-rs/lexer.html) ~ 20 行代码
- [解释器](https://vishpat.github.io/lisp-rs/parser.html) ~ 60 行代码
- [评估器](https://vishpat.github.io/lisp-rs/evaluator.html) ~ 170 行代码
- [REPL](https://vishpat.github.io/lisp-rs/repl.html) ~ 30 行代码

了解解释器实现的最佳方法是在阅读本文档时检查代码并逐步浏览它。

```bash
git clone https://github.com/vishpat/lisp-rs
git checkout 0.0.1
```

一旦您彻底理解了实现，您将能够向其添加新功能，例如支持新数据类型（如字符串、浮点数、列表）或函数式编程结构（如映射、过滤器、化简函数等）。

### REPL
运行解释器并获取其 REPL (Read-Eval-Print-Loop)

```bash
cargo run
```

