---
title: "Conda"
date: 2024-08-18T19:20:05+08:00
draft: false
tags:
  - python
  - conda
---

## conda 使用指南

miniconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda

更改仓库地址: https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

## 环境管理

```shell
conda create --name myenv python=3.9
conda activate myenv
conda deactivate

conda env list

conda remove --name myenv --all
```

## 包管理

```shell
conda install requests

conda list

conda update requests
conda update all

conda list --explicit > a.txt
conda install --file a.txt
```
