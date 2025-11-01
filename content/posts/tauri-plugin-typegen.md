---
title: "Tauri Plugin Typegen"
date: 2025-10-31T19:44:37+08:00
draft: false
tags:
  - rust
  - tauri
---

> BUG真的是太多了，略复杂一点的都搞不定，还是手动使用 [ts-rs](https://github.com/Aleph-Alpha/ts-rs) 生成比较好
> 
> 虽然今天终于上传代码到github上，[tauri-typegen](https://github.com/thwbh/tauri-typegen)，并且还改了名称，
> 可以关注下，期待稳定下来

`tauri-plugin-typegen` 确实是个好东西，可以为tauri注册的command及返回值自动生成typescript代码。
虽然有一些bug，但仍然是可以用的，官方文档及命令行都有一些问题。

## 命令行使用

```shell
cargo install tauri-plugin-typegen
```

> 当前是 `0.1.5` 版本


然后在项目根目录下执行，`cargo tauri-typegen -v none`，将会在 src/generated 中生成 typescript 文件,
但是要注意，下次执行时，并不会覆盖，需要先手动删除，再执行命令

## 放到build.rs中自动执行

```shell
cargo add --build tauri-plugin-typegen
```

然后修改build.rs，如下

```rust
use tauri_plugin_typegen::{GenerateConfig, generate_from_config};

fn main() {
    println!("cargo:rerun-if-changed=tauri.conf.json");

    let _ = std::fs::remove_dir_all("../src/generated"); // 注意先删除了

    generate_from_config(&GenerateConfig {
        project_path: ".".to_string(),
        output_path: "../src/generated".to_string(),
        validation_library: "none".to_string(),
        verbose: Some(true),
        ..Default::default()
    })
    .expect("Failed to generate TypeScript bindings");

    tauri_build::build()
}
```

官方文档的示例虽然简单，但是编译不过，主要是会生成 `println!("cargo:rerun-if-changed=./src-tauri");`
导致编译时报错。

## 其他注意事项

- validation 默认是 zod，但我认为rust返回对象肯定是对，没有必要validate
- 依赖并不是很好追踪，如果想就想重新编译，可以 `touch tauri.conf.json`
