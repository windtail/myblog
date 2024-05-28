---
title: "Rust minimize"
date: 2024-05-28T09:50:31+08:00
draft: false
tags:
  - rust
---

rust默认的release版仍然挺大的，其实还有改进空间，[这里](https://github.com/johnthagen/min-sized-rust?tab=readme-ov-file)有介绍，
[这里](https://doc.rust-lang.org/cargo/reference/profiles.html)有官方文档。简单来说可以在`cargo.toml`中添加以下内容

```toml
strip = true
opt-level = "s"
lto = true
panic = "abort"
```

编译完成后，还可以继续使用 [upx](https://github.com/upx/upx) 来压缩体积，但是有时候不好使，所以这个得看缘份。
