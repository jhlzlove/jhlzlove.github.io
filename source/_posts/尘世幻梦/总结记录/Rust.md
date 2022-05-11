---
title: Rust
categories: rust
abbrlink: a49987e3
---

Rust浅尝辄止。官网教程链接：https://www.rust-lang.org/zh-CN/learn

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=true} -->

## Rust的基本规则

Rust命名规范：多个单词使用下划线隔开，例如：`hello_world`
Rust的缩进是4个空格，不是 `tab`
代码以分号结尾
运行 Rust 之前必须先编译，`rustc 源文件名.rs`;`rustc` 只适合简单的 rust 程序。
在 Rust 里，第三方的包称为 crate

## cargo

cargo 是 Rust 的构建系统和包管理器，类似 Python 的 pip 和 Java 的 Maven 或者 Gradle；
如果项目不是使用 `cargo` 创建的，也可以改变目录结构为 cargo 创建的：

1. 把源代码都移动到 `src` 下；
2. 创建 `Cargo.toml` 并填写相应的配置。

`cargo build`:创建可执行文件，win下是 `*.exe`,Linux 无后缀。运行可执行文件即运行程序。
第一次执行 cargo build 命令会在顶层目录生成 `Cargo.lock` 文件。该文件负责追踪项目依赖的精确版本，不需要手动修改该文件，也不要去修改它！

`cargo run`: 构建并且运行项目；如果之前该项目编译过且没有修改内容，再次执行该命令直接运行而无须编译。

`cargo check`: 检查代码，确保能通过编译，但不产生任何可执行文件。速度比 cargo build 快得多。编写代码时可以周期性使用改命令去检查代码，提高效率。

为发布构建，项目上线时可以使用。
`cargo build --release`: 编译时会进行优化，代码会运行的更快，但是编译时间更长，会在 `target/release` 而不是 `target/debug` 生成可执行文件。

尽量使用 `cargo`

## 项目升级新版本

基础迁移：

1. 将旧的项目迁移到新版本时，可以使用 cargo 来自动修复原本项目文件中不兼容的代码：
`cargo fix --edition`
但是，`cargo fix` 不能总是自动修复这些不兼容的代码。如果它不能修复的话，将会吧错误信息打印到控制台，这部分代码需要我们手动去更新。
2. 然后，编辑 `Cargo.toml` 并将 `edition` 字段设置更新的版本，例如edition = "2021"
3. 最后，使用 `cargo test` 验证修复是否有效.

高级迁移移步官网：https://doc.rust-lang.org/edition-guide/editions/advanced-migrations.html

### 代码编译

rustc hello.rs

### rustdoc 生成文档

rustup doc:本地帮助文档。

使用 rustdoc 生成文档时，注意，最好在项目的根路径下生成，不然它会在你调用命令的当前路径下生成。

在任何给定的时刻，只能满足下列条件之一：

- 一个可变的引用；
- 任意数量不可变引用；
引用必须一直有效。

不支持所有权的数据类型：切片（slice）

Rust 的所有权规则：

- 一次只能有一个所有者
- Rust 中的每一个值都有一个变量，称其为 owner
- 当所有者超出范围时，该值将被删除。
