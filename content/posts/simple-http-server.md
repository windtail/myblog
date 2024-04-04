---
title: "Simple Http Server"
date: 2024-04-04T21:29:42+08:00
draft: false
tags:
  - rust  
---

调试时，简单的http服务，以前一直使用

```shell
python -m http.server
```


但是有些[脚本](https://pyscript.github.io/docs/2024.3.2/user-guide/workers/)需要cors头，
python默认就不行了，还得自己写。不过，幸好还有rust的[simple-http-server](https://crates.io/crates/simple-http-server)

```shell
cargo install simple-http-server
```

```shell
simple-http-server.exe --coep --coop --cors
```
