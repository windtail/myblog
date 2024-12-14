---
title: "Uv Python Versions"
date: 2024-12-14T22:06:37+08:00
draft: false
tags:
  - python  
  - uv
---

## uv python version management

[uv](https://docs.astral.sh/uv/)在管理依赖上虽不如poetry用得爽，但是人家还管理pyenv和pipx类似的功能，还是挺好用的。
比如今天安装labelImg时，python 3.12 运行会出问题，然后我就是需要一个 python 3.8

```shell
uv python list --all-versions
```

找到其中的3.8最新版本，现在是 cpython-3.8.20-windows-x86_64-none，然后执行

```shell
uv python install cpython-3.8.20-windows-x86_64-none
```

## uv install tool like pipx

```shell
uv tool install labelimg -p 3.8
```

话说 uv 真的很快，秒杀 poetry