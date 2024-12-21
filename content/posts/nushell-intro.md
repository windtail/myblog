---
title: "NuShell Intro"
date: 2024-12-20T13:20:38+08:00
draft: false
tags:
  - nushell
  - shell
---

## 简介

[NuShell](https://www.nushell.sh) 是一个用rust写的跨平台的shell。

NuShell最重要的一点是引入了`结构化数据`的概念，命令之间传递的不再是纯文本，而List/Record/Table等等。
这是一个很大的改变，这意味着以前经常要用到grep/sed/awk/cut等命令，现在基本上不会再用了，
不需要去记各种小语言的语法了，一旦学会了如何操作 List/Record/Table，那么所有返回这些复杂对象的命令，
处理起来都是一样的。

其次重要的概念是`管道`，它是NuShell的精髓，它使得命令的组合变得非常容易，看起来就像是数据的流水线，
非常地自然。

## 相比 Bash

Bash本身的语法并不是很复杂，但是Bash本身的命令并不是很全，多数是靠外部命令来完成的，
所以我们还得再学习其他命令的语法，尤其是sed和awk

Bash各命令处理的都是字符串，而NuShell有很多数据类型，
integer, float, boolean, string, path, directory, list, record, table,
closure, glob等等，所以NuShell更像是一门专业的编程语言，编程比Bash强很多。

NuShell是rust写的，很多语法很像rust，并且在运行前就可以根据类型静态检查语法，
相当于在编译时就能报出很多错误。

## 相比 [Xonsh](https://xon.sh/)

Xonsh也是一个很好的项目，可以把外部命令调用和Python代码混着写，借助了Python强大的生态，
可以写出任意复杂的shell脚本，但是它没有内置常用的命令，导致rm/cp/mv等等在Windows上用不了，
要想真的跨平台，多数时候都是直接写Python，用shutil包。这就会让简单的脚本，看着也有点复杂了。

NuShell就好很多，安装时只有一个exe文件，就支持了非常多的内置命令，跨平台写脚本时，太方便了。

## 文档丰富

[官方文档](https://www.nushell.sh/book/)非常得好，我这里只简单说几个重要的。


### completions

```shell
git clone github:nushell/nu_scripts.git
```

`custom-completions` 文件夹中支持很多命令补全。可以将需要的命令添加 `$nu.config-path` 文件中，
例如：

```text
use /path/to/nu_scripts/custom-completions/git/git-completions.nu *
use /path/to/nu_scripts/custom-completions/poetry/poetry-completions.nu *
use /path/to/nu_scripts/custom-completions/cargo/cargo-completions.nu *
use /path/to/nu_scripts/custom-completions/rustup/rustup-completions.nu *
use /path/to/nu_scripts/custom-completions/scoop/scoop-completions.nu *
use /path/to/nu_scripts/custom-completions/pnpm/pnpm-completions.nu *
use /path/to/nu_scripts/custom-completions/ssh/ssh-completions.nu *
```

### 环境变量

环境变量应该是 `$nu.env-path` 中添加，例如

```text
$env.RUSTUP_UPDATE_ROOT = 'https://mirrors.tuna.tsinghua.edu.cn/rustup/rustup'
$env.RUSTUP_DIST_SERVER = 'https://mirrors.tuna.tsinghua.edu.cn/rustup'
```

### 安装插件

```shell
let plugins = [ nu_plugin_inc
  nu_plugin_clipboard
  nu_plugin_port_list
  nu_plugin_hashes
  nu_plugin_gstat
  nu_plugin_formats
  nu_plugin_query
] 

$plugins | each { cargo install $in --locked } | ignore
$plugins  | each { which $in | get 0.path | plugin add $in } | ignore
```

安装完成后，重新打开一个shell，就可以使用这些插件了。
