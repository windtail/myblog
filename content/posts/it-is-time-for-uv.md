---
title: "It Is Time for Uv"
date: 2025-05-25T09:55:11+08:00
draft: false
---

[uv](https://docs.astral.sh/uv/) 确实是一个很快很快的包管理工具，最重要的是它不止一个包管理工具，
还可以管理python的不同版本！还集成pip，pipx！

## Python版本管理

Python最近一段时间发展很快，各版本之间多有不兼容，为了用一些新功能，常常需要规定版本，以便出现不兼容的情况。
只要在工程的根目录新建一个 `.python-version` 文件，里面写上版本号，如 `3.11.10` ，然后执行 `uv sync`，就会自动下载对应的版本，
并自动完成所有packages的安装。默认uv会从github的[release](https://github.com/astral-sh/python-build-standalone/releases)下载，好在这里有[镜像](https://mirror.nju.edu.cn/github-release/indygreg/python-build-standalone/)，
但是这个镜像只同步了最新的Python，如果需要下载旧版本Python，就需要连github了。镜像可以使用环境变量设置

```
UV_PYTHON_INSTALL_MIRROR = 'https://mirror.nju.edu.cn/github-release/indygreg/python-build-standalone/'
```

## Migrate to uv

如果你有一个其他工具管理的Python工程，现在你只需要执行 `uvx migrate-to-uv`，你就会得到一个配置好的 pyproject.toml ！！

## Install

参考[官方文档](https://docs.astral.sh/uv/getting-started/installation/)，Windows建议使用scoop安装

```shell
scoop install uv
```

安装完成后，应设置pypi镜像，可通过设置环境变量

```
UV_DEFAULT_INDEX = 'https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple'
```
