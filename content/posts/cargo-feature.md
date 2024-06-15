---
title: "Cargo Feature"
date: 2024-06-15T21:26:25+08:00
draft: false
tags:
  - rust
---

`cargo add` 确实挺好用，但是某个crate到底有多少个`feature`呢，经常连文档都说得不是很清楚。
幸好我们有 `cargo feature` 可以列出某个crate的所有feature

```shell
cargo install --locked cargo-feature
```

> 注意不是 `features`
