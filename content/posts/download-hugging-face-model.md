---
title: "Download Hugging Face Model"
date: 2024-10-04T20:41:41+08:00
draft: false
tags:
  - ai
---

## 国内镜像

https://hf-mirror.com/

## 具体步骤

创建conda环境，如果不记得，可以参考[这里]({{< ref "/posts/conda.md" >}})

```shell
conda create --name hfd python=3.12.5
conda activate hfd
conda install huggingface_hub
```

```shell
export HF_ENDPOINT=https://hf-mirror.com    # bash
$env:HF_ENDPOINT = "https://hf-mirror.com"  # pwsh
```

```shell
huggingface-cli download --resume-download gpt2 --local-dir gpt2  # 下载模型，gpt2为模型名称
huggingface-cli download --repo-type dataset --resume-download wikitext --local-dir wikitext  # 下载数据集
```

## 使用golang编译的单个exe来完成

官方的是这个 https://github.com/bodaay/HuggingFaceModelDownloader ，
但是官方不支持mirror，我改了下，但是官方没理我，我的在[这里](https://github.com/windtail/HuggingFaceModelDownloader)
