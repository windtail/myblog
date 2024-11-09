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
- 将默认的venv的路径设置到工程根目录中的 `.venv`：`poetry config virtualenvs.in-project true`
- 如果C盘空间不足，可以将cache设置到D盘：`poetry config cache-dir D:/.poetry`

## [常用命令](https://python-poetry.org/docs/cli/)

- 新建工程：`poetry new xxx`
- 在已有工程中添加：`poetry init`
- 添加依赖：`poetry add <dep>`
- 删除依赖：`poetry remove <dep>`
- 手动修改了`pyproject.toml`后，更新 `poetry.lock` 文件：`poetry lock --no-update`，注意`--no-update`否则会所有依赖都更新了
- 安装依赖：`poetry install`
- 只安装运行时依赖：`poetry install --only main`
