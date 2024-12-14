---
title: "Poetry Pytorch"
date: 2024-12-13T15:13:37+08:00
draft: false
tags:
  - python
  - poetry
---

## 问题

最近试了试 [yolo11](https://docs.ultralytics.com/)，直接安装

```shell
poetry add ultralytics
```

这样下来的是 cpu 版本的 torch，在执行下面的代码时，会报torch编译时没有使能cuda

```shell
model = YOLO("yolo11n.pt")
model.to("cuda")
```

## 解决方案

解决方案在[这里](https://github.com/python-poetry/poetry/issues/6409)

首先要安装cuda toolkit，然后查看下版本，最后在[这里](https://pytorch.org/get-started/locally/)看下pip index应该用什么url，
最后执行下面的命令即可，torch/torchvision/torchaudio 都会被替换为 cuda 版本。

```shell
poetry source add --priority=explicit pytorch https://download.pytorch.org/whl/cu124
poetry add torch torchvision torchaudio --source pytorch
```

安装完成后，可以用如下代码检查是否安装成功

```shell
import torch
print(torch.cuda.is_available())
print("CUDA Version:", torch.version.cuda)
```

## 其他

[这里](https://docs.infini-ai.com/posts/download-pytorch-from-mirror.html)说的，阿里的pytorch wheels，
是不能用的，因为它没有提供完整的index服务，所以poetry没法直接用，会报找不到package