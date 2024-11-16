---
title: "step-ca"
date: 2024-11-16T19:19:09+08:00
draft: false
tags:
  - ca
  - docker
---

当我们本地需要运行一组服务器时，还需要使用https时，我们就需要手动地签发好多证书。
有没有一种可能，我们可以在本地跑一个像 `Lets Encrypt` 的服务？

还真有，在这里 [step-ca](https://hub.docker.com/r/smallstep/step-ca)，首先我们
需要一个设置一个dns服务，这个可以用 [strm/dnsmasq](https://hub.docker.com/r/strm/dnsmasq/)，
然后就可以使用 ACME 的客户端来申请证书了。

这是 docker-compose.yml 文件内容，其中`./password.txt` 里应该存一个比较长的随机密码

```yaml
version: '3.8'
services:
  step-ca:
    image: smallstep/step-ca:latest
    secrets:
      - ca_password
    environment:
      - DOCKER_STEPCA_INIT_NAME=YourCaName
      - DOCKER_STEPCA_INIT_DNS_NAMES=Your.Ca.Server.Domain.Name
      - DOCKER_STEPCA_INIT_REMOTE_MANAGEMENT=true
      - DOCKER_STEPCA_INIT_ACME=true
      - DOCKER_STEPCA_INIT_PASSWORD_FILE=/run/secrets/ca_password
    volumes:
      - step:/home/step
    ports:
      - "9000:9000"  
    logging:
      driver: "local"
      options:
        max-size: "256m"
    restart: always
    sysctls:
      net.core.somaxconn: 16384
      net.ipv4.tcp_max_syn_backlog: 8192
      net.ipv4.tcp_tw_reuse: 1
      net.ipv4.tcp_timestamps: 1
      net.ipv4.ip_local_port_range: 1024 65535
    ulimits:
      nproc: 65535
      nofile:
        soft: 20000
        hard: 40000
secrets:
  ca_password:
    file: ./password.txt

volumes:
  step:
    driver: "local"
```

自动生成的证书在 `https://Your.Ca.Server.Domain.Name:9000/roots.pem` 下载，应该把这个证书下载下来，
然后添加到信任。ACME服务在 `https://Your.Ca.Server.Domain.Name:9000/acme/acme/directory`