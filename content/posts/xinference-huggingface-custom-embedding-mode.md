---
title: "Xinference Huggingface Custom Embedding Model"
date: 2025-01-15T19:38:29+08:00
draft: false
---

## [Xinference](https://inference.readthedocs.io/zh-cn/latest/) 

Xinference 是一个开源的 LLM 服务框架，它提供了一套完整的 LLM 服务，包括模型加载、模型推理、模型管理、模型部署等。

为了方便部署，这里采用docker运行

```shell
docker pull registry.cn-hangzhou.aliyuncs.com/xprobe_xinference/xinference:v1.2.0
docker tag registry.cn-hangzhou.aliyuncs.com/xprobe_xinference/xinference:v1.2.0 xprobe/xinference:v1.2.0
```

为了使用gpu，还需要安装 [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installing-with-apt)，

```shell
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker  # 这将会修改 /etc/docker/daemon.json
sudo systemctl restart docker   
```

可以使用如下命令测试下，注意ubuntu版本

```shell
docker run --rm --gpus all nvidia/cuda:12.6.3-base-ubuntu24.04 nvidia-smi
```

```shell
docker run --name xinference --rm -e XINFERENCE_MODEL_SRC=modelscope \
  -e HF_ENDPOINT=https://hf-mirror.com \
  -v ./data/.xinference:/root/.xinference \
  -v ./data/.cache/huggingface:/root/.cache/huggingface \
  -v ./data/.cache/modelscope:/root/.cache/modelscope \
  -p 9997:9997 --gpus all xprobe/xinference:v1.2.0 xinference-local -H 0.0.0.0 --log-level debug
```

这个时候可以访问下9997端口，就可以使用xinference的webui了，左下角可以配置语言为中文


## 注册一个自定义模型

下载 hf-mirror 的专用下载工具

```shell
wget https://hf-mirror.com/hfd/hfd.sh
chmod a+x hfd.sh
sudo apt install aria2
cd data/.cache/huggingface
./hfd.sh TencentBAC/Conan-embedding-v1
```

下载完成后，在webui中，左侧找到注册模型，上面选嵌入模型，然后填入相关参数，
这些参数我暂时还没有找到在哪里获取，可能是模型中的config.json文件，但是好像不太全，
这个模型我是在这里获得到 https://huggingface.co/spaces/mteb/leaderboard

## 启动自定义模型

注册了之后，在webui左边，选择启动模型=>自定义模型=>嵌入模型，选择然后就可以启动了

可以用以下的langchain代码来测试

```python
from langchain_community.embeddings.xinference import XinferenceEmbeddings

embeddings = XinferenceEmbeddings(
    server_url="http://localhost:9997",
    model_uid ="Conan-embedding-v1" # replace model_uid with the model UID return from launching the model
)

embeddings.embed_query("你好呀")
```
