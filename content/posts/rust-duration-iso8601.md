---
title: "Rust ISO8601"
date: 2025-10-19T13:25:50+08:00
draft: false
tags: [ "rust", "iso8601" ]
---

`ISO8601` 格式的 DateTime 和 Duration 经常会在 REST API 中遇到，rust 标准库支持得不是很好，
经常问AI，这回记录一下

```toml
time = { version = "0.3.44", features = ["macros", "serde", "serde-well-known"] }
iso8601-duration-serde = "0.1"
```

```rust
#[derive(Serialize, Deserialize)]
pub struct Info {
    #[serde(with="time::serde::iso8601")]
    last_modified_time: time::OffsetDateTime,
    
    #[serde(with="iso8601_duration_serde")]
    cycle_time: time::Duration,    
}
```
