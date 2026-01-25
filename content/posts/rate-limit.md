---
title: "Rate Limit"
date: 2026-01-25T19:36:15+08:00
draft: false
tags:
  - rust
---

rust出错的时候，需要log一下，但是又不能太快，以防打太多错误日志，
可以简单地使用`rate_limit_macro::rate_limit`

```rust
rate_limit!(rate = 1, interval = 10, {
    log::error(...);
});
```
