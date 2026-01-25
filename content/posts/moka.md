---
title: "Moka"
date: 2026-01-25T10:48:50+08:00
draft: false
tags:
  - rust
---

[Moka](https://crates.io/crates/moka)是一个cache，支持异步和同步，并且是ThreadSafe的，
更好的是，读的时候是无锁的，写的时候锁也很小。并且还支持

- Time to live：时间到了，就删除
- Time to idle：如果这么长时间没有访问，就删除
- Per-entry variable expiration：某一个可以单独处理
- maximum number of entries：总数到了就删除一些
- total weighted size of entries：总大小到了就删除一些

如果自己要搞一个rwlock+hashmap，这些东西还挺复杂的，所以直接用就好了。
