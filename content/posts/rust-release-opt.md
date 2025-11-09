---
title: "Rust Release Opt"
date: 2025-11-09T12:44:38+08:00
draft: false
tags: ["rust"]
---

## release Cargo.toml

```toml
[profile.release]
strip = true
opt-level = "s"
lto = 'fat'
codegen-units = 1
panic = "abort"
```

## debug only

一些调试时使用的代码，可以用 `#[cfg(debug_assertions)]` 来包裹，
debug时的检查可以用 `debug_assert!`

```rust
#[cfg(debug_assertions)]
debug_only_code();

debug_assert!(xxx)
```

## features

根据不同使用场景，可以使用feature选择不同的代码和写法。可以在 Cargo.toml 声明不同的feature

```toml
[features]
feature1 = []
feature2 = []
```

代码中使用

```rust
#[cfg(feature = "feature1")]
fn feature1_code() {}

#[cfg(feature = "feature2")]
fn feature2_code() {}
```

## target os

如果代码是跨平台的，那么不同的操作系统可以使用不同的代码

```rust
#[cfg(target_os = "linux")]
fn linux_code() {}

#[cfg(target_os = "windows")]
fn windows_code() {}
```
