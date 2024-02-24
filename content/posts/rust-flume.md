---
title: "Rust Flume"
date: 2024-02-24T20:08:59+08:00
draft: false
tags:
  - rust
  - channel
---

`“Do not communicate by sharing memory; instead, share memory by communicating.”`

## flume

习惯了Go的channel，经常需要在不同的线程和异步任务中传数据。rust也有，同步异步之间都可以传，
还可以unbound，真是不错。每次想起channel时，总是先想起 `crossbeam-channel`，
但是实际我是想用 [flume](https://docs.rs/flume/latest/flume/)，记录下，以防下次再忘记。

## example

文档中的示例，贴一个

```rust
let (tx, rx) = flume::unbounded();

tx.send(42).unwrap();
assert_eq!(rx.recv().unwrap(), 42);
```
