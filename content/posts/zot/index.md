---
title: "Zot"
date: 2026-01-31T16:57:34+08:00
draft: false
tags:
  - docker
---

## docker registry

本地需要管理一些docker镜像时，可以使用官方的[registry](https://hub.docker.com/_/registry)，
但官方的registry不支持管理，查看、删除都费劲。

[zot](https://github.com/project-zot/zot)这个项目不错，部署起来也很简单，并且支持权限管理。

```yaml
services:
  zot:
    image: ghcr.io/project-zot/zot-linux-amd64:v2.1.14
    restart: always
    environment:
      - VIRTUAL_HOST=your.host.com # 如果使用jwilder/nginx-proxy，需要配置
      - VIRTUAL_PORT=5000
    volumes:
      - ./zot/config.json:/etc/zot/config.json
      - ./zot/htpasswd:/etc/zot/htpasswd
      - /srv/zot:/var/lib/zot
```

### config.json

config.json参考[这里](config.json)，注意 `"compat": ["docker2s2"]`，这项配置是为了兼容docker v2格式的镜像。

### htpasswd

htpasswd就是原来apache的那个密码文件，密码文件可以使用`htauth-cli`这个工具

```bash
cargo install htauth-cli
```

### nginx 额外配置

```text
client_max_body_size 0;
proxy_request_buffering off;
proxy_buffering off;
proxy_read_timeout 900;

proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Port  $server_port;
proxy_set_header Host $http_host;
```
