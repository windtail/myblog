---
title: "Cargo Ssh Git"
date: 2024-01-28T19:11:16+08:00
tags:
  - rust
  - cargo
  - git
draft: false
---

很多时候rust都依赖一些很新库，直接引用了git，这些git网址一般是https，
但是我一般都想使用ssh来连接。

## Git 配置

`~/.gitconfig`中添加

```
[url "github:"]
	insteadOf = https://github.com/
```

> `github`是`.ssh/config`中的`HostName`

## Cargo 配置

cargo [不会](https://github.com/rust-lang/cargo/issues/2078) 读取 `.ssh/config`

但是，可以添加配置，使用git命令行来获取。在 `~/.cargo/config` 中添加

```toml
[net]
git-fetch-with-cli = true
```
