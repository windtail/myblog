---
title: "Docker Proxy"
date: 2024-11-17T16:08:06+08:00
draft: false
tags:
  - docker
  - proxy
---

在命令行中输入 `docker` 命令会涉及多少个代理阶段呢？答案是3个

1. **docker -> docker daemon**

   docker是一个客户端，它可以通过网络访问多个daemon，不一定是本机。docker在访问daemon时会使用标准的`$http_proxy/$https_proxy`环境变量

2. **docker daemon -> docker hub/other**

   docker daemon访问外网时，如果需要proxy，需要[这样](https://docs.docker.com/engine/daemon/proxy/#daemon-configuration)配置。
   ```json
    {
      "proxies": {
        "http-proxy": "http://proxy.example.com:3128",
        "https-proxy": "https://proxy.example.com:3129",
        "no-proxy": "*.test.example.com,.example.org,127.0.0.0/8"
      }
    }
    ```

> 以上配置没有用，需要更改systemd的配置，在相同的页面也有介绍：
> 新建文件 `/etc/systemd/system/docker.service.d/http-proxy.conf`，并写入以下内容
> ```text
> [Service]
> Environment="HTTP_PROXY=socks5://127.0.0.1:1080" "HTTPS_PROXY=socks5://127.0.0.1:1080"
> ```

3. **docker container**

   如果`docker build`和`docker run`时，容器自己还可能需要访问网络，这个在[这里](https://docs.docker.com/engine/cli/proxy/#configure-the-docker-client)配置。需要创建一个 `~/.docker/config.json`
   ```json
    {
     "proxies": {
       "default": {
         "httpProxy": "http://proxy.example.com:3128",
         "httpsProxy": "https://proxy.example.com:3129",
         "noProxy": "*.test.example.com,.example.org,127.0.0.0/8"
       }
     }
    }
    ```

注意 这两个json文件是不太一样的，尤其是一个是 snake-case，另一个是 camelCase
