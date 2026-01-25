---
title: "Mimalloc"
date: 2026-01-25T10:25:49+08:00
draft: false
tags:
  - rust
  - alloc
---

## 使用方法

使用的rust在容器中运行，或需要静态链接（到处可用）时，经常喜欢使用musl，但据说musl默认的动态
内存分配器性能非常地差，应该使用`mialloc`替换一下，否则，尤其是异步，很容易踩坑。

具体步骤也很简单的

```shell
cargo add mialloc
```

然后随便找个文件，添加如下代码

```rust
use mialloc::MiMalloc;

#[global_allocator]
static GLOBAL: MiMalloc = MiMalloc；
```

## 注意事项

如果在写一个library crate，应该使用feature处理一下，
因为rust只允许一个全局分配器，如果库中写死了，那么binary crate就没有选择权，
万一想用别的allocator，就会导致编译报错。

