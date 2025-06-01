---
title: Rust 写脚本？可以的！（上）
date: 2025-05-31 20:05:38
cover:
categories:
  - 折腾
  - 教程
  - 技术
tags:
  - Rust
  - 脚本
---

Rust 在很多情况下都被认为是个优秀的语言... 但是，拿 Rust 写脚本？其实是可行的！拜 rust-script 项目所赐，Rust 也可以是一个脚本语言。

<!--more-->

## 起因

最近我在计划重写我的博客（敬请期待）和对应的基于 GitHub Issue 的友链生成脚本。由于我前段时间在学习 Rust 的时候发现了 [rust-script](https://rust-script.org/) 这么一个项目，而且对其很感兴趣，于是决定拿这个折腾一下。

## rust-script

首先，简单介绍一下 rust-script。[rust-script](https://rust-script.org/)（[GitHub](https://github.com/fornwall/rust-script)）是一个 Rust 库，其允许用户使用 Rust 编写脚本，并通过 Unix 风格的 shebang 语法或者 Windows 文件关联来运行 Rust 脚本。

实际上，rust-script 并不是一个完整的 Rust 解释器。其本质仍然是将对应的脚本的 Cargo 项目编译成对应平台的可执行文件，然后运行这个可执行文件。虽然对于代码规模较大的脚本来说，初次编译代码仍然需要一些时间，但由于 Rust 会缓存编译结果，所以当脚本被多次运行时，编译时间仍然是可以接受的。而且这样让一切都维持普通 Rust 项目的形式（编译 Cargo 项目并运行对应的可执行文件）也带来了一个优势：保证了 Rust 代码的兼容性，即我们不需要担心一些依赖因为没有在其设计环境内使用而造成问题。

## rust-script 的使用

要使用 rust-script 很简单。首先，使用 Cargo 安装 rust-script：

```bash
cargo install rust-script
```

注意，必须使用 Rust 1.74 或更新版才可以正常使用 rust-script。

对于 macOS 用户，rust-script 还提供了一个 Homebrew 打包：

```bash
brew install rust-script
```

安装完成后，你就可以使用 `rust-script` 命令来运行 Rust 脚本了。

## 编写第一个 Rust 脚本

首先，我们来写一个 Hello World。创建一个名为 `hello.rs` 的文件，内容如下：

```rust
fn main() {
    println!("Hello, world!");
}
```
然后，在终端中运行以下命令：

```bash
rust-script hello.rs
```
如果一切正常，你应该会看到输出：

```
Hello, world!
```

但实际上，这个脚本还可以更简单一些，例如：省略掉 `fn main()` 函数，直接写入代码：

```rust
println!("Hello, world!");
```

在这种情况下，rust-script 会自动将代码包装在 `fn main() { ... }` 中并运行。

## 依赖

Rust 拥有非常丰富的生态系统，而我们通常也会希望可以在脚本中使用一些依赖。如要在脚本中使用依赖，我们可以在脚本开头添加 `//!` 注释，并使用 Markdown 语法添加一个 `cargo` 语言的代码块，然后使用 `cargo.toml` 内会使用的语法格式来添加依赖。例如：

```rust
//! This is a regular crate doc comment, but it also contains a partial
//! Cargo manifest.  Note the use of a *fenced* code block, and the
//! `cargo` "language".
//!
//! ```cargo
//! [dependencies]
//! time = "0.1.25"
//! ```
fn main() {
    println!("{}", time::now().rfc822z());
}
```

这样，在运行的时候，rust-script 会自动解析这个代码块，并将其作为 `Cargo.toml` 的内容来处理。也就是说，其内部的依赖会在编译过程中被包含。

## Shebang 与 Windows 文件关联

最后，如果每次要执行脚本都要输入 `rust-script` 命令，还是有点麻烦的。我们可以使用 Unix 风格的 shebang 语法来让脚本可以直接运行。
在脚本的第一行添加以下内容：

```rust
#!/usr/bin/env rust-script
```

这样，你就可以直接运行脚本了。例如：

```bash
chmod +x hello.rs
./hello.rs
```

实际上，如果 shebang 被加入到了脚本中，该脚本文件甚至不需要有 `.rs` 后缀名。你可以将其重命名为 `hello`，然后直接运行：

```bash
chmod +x hello
./hello
```

对于 Windows 用户，rust-script 项目官方的建议是使用 `.ers` 后缀名（即 executable Rust - 可执行 Rust），并将其与 `rust-script` 关联。这样，脚本就可以直接双击运行了。

## 预告

在下一篇文章中，我将会简单介绍一下我发现的一些编写 rust-script 脚本时可以使用到的开发环境配置技巧，以及如何在 GitHub Action 中运行 rust-script 脚本。下次再见！
