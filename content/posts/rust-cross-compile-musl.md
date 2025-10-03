---
title: "Rust Cross Compile Musl"
date: 2025-10-03T21:39:04+08:00
draft: false
tags: ["rust", "cross compile", "musl"]
---

# Why musl

Rust 编译的程序默认是依赖 glibc 的，这会导致在新的系统上编译的 rust 程序，无法在旧的系统器上运行，有一些服务器运行了一些很老版本的系统，
如 debian stretch 之类的，不可能为一个命令行工具更新整个系统，这样会导致服务器上稳定运行的服务需要更新。

musl 是一个兼容 glibc 的 C 语言标准库，体积较小，可以直接静态编译，运行在任意系统上。

# How to 

借助于 [cross crate](https://crates.io/crates/cross)，我们在安装完成 `docker` 和 `cargo install cross` 之后，
可以运行 `cross build --target x86_64-unknown-linux-musl --release` 来编译一个静态链接的 rust 程序。
