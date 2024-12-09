---
title: "Poetry"
date: 2024-11-09T21:42:38+08:00
draft: false
tags:
  - python
---

## 介绍

曾经有一段时间想用 `pipenv` ，但后来因种种原因放弃了，后面就一直使用 requirements.txt，
直到最近用了 `pyproject.toml` + `flit` 的方案，但总是感觉不太好，不如 `cargo` `pnpm` 之类的工具用得爽。

最近在[同事](https://rustcc.com.cn/)的介绍下，尝试了下 `poetry`，感觉真是挺好的，pycharm也支持得很好。
和 `cargo` 一样好用了。

## [安装](https://python-poetry.org/docs/#installation)

建议使用 `pipx` 安装

## [配置](https://python-poetry.org/docs/configuration/)

- 查看当前配置： `poetry config --list`
- ~~将默认的venv的路径设置到工程根目录中的 `.venv`：`poetry config virtualenvs.in-project true`~~ (放在外面还是好一点，省得搜索时总搜到这里的代码)
- 如果C盘空间不足，可以将cache设置到D盘：`poetry config cache-dir D:/.poetry`

## [常用命令](https://python-poetry.org/docs/cli/)

- 新建工程：`poetry new xxx`
- 在已有工程中添加：`poetry init`
- 使用指定的python（[官方](https://python-poetry.org/docs/managing-environments/#switching-between-environments)）：`poetry env use /full/path/to/python`
- 添加依赖：`poetry add <dep>`
- 添加dev依赖：`poetry add <dep> -G dev`
- 添加某个平台的依赖：`poetry add <dep> --platform win32`
- 删除依赖：`poetry remove <dep>`
- 手动修改了`pyproject.toml`后，更新 `poetry.lock` 文件：`poetry lock --no-update`，注意`--no-update`否则会所有依赖都更新了
- 安装依赖：`poetry install`
- 只安装运行时依赖：`poetry install --only main`

### extras

poetry对extras支持得并不是很好，本以为group会自动映射成extras，但实际并不是这样。group只能被poetry安装，extras是pip安装时要的东西，
需要将某个package设置为optional，例如 `pygit2 = { version = "^1.16.0", optional = true }`

然后添加
```toml
[tool.poetry.extras]
extra_name = ["pygit2", "other-package"]
```

默认poetry install时还不会安装，需要手动安装，例如 `poetry install --extras extra_name` 或者 `poetry install --all-extras`

### export to requirements.txt

poetry可以导出为requirements.txt（[官方文档](https://python-poetry.org/docs/cli/#export)），然后就可以用pip安装了。

```shell
poetry export -f requirements.txt --output requirements.txt
```


## 添加pypi镜像

如果只是为本工程添加镜像，可以 `pyproject.toml` 添加如下行

```toml
[[tool.poetry.source]]
name = "tsinghua"
url = "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
priority = "primary"
```

但是全局就没有那么友好了。还得装个[插件](https://github.com/arcesium/poetry-plugin-pypi-mirror)，按插件说明

```shell
pipx inject poetry poetry-plugin-pypi-mirror
```

手动打开poetry用户配置文件，注意，没法使用`poetry config`命令，用户配置文件在这里

```shell
notepad $env:APPDATA\pypoetry\config.toml
```

在打开的文件中添加

```toml
[plugins]
[plugins.pypi_mirror]
url = "https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple"
```

## 题外话

我现在更喜欢的镜像是 https://pypi.cstcloud.cn/simple/ 速度挺快的！
