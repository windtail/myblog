---
title: "Gitlab Ci Docker Auth"
date: 2025-11-30T16:20:50+08:00
draft: false
tags:
  - docker
  - gitlab
  - cicd
---

## Gitlab CI docker

### 问题

Gitlab CI使用docker比较方便，但是现在docker越来越不容易拉取了，并且官方确实是太慢了。
阿里云的ACR个人版是免费的，访问速度快，适合CI/CD使用，最重要的是免费的！

但是问题来了，2024年9月之后不再支持免密拉取，必须要使用密码才能拉取。Gitlab CI使用的镜像拉取时，
如何自动login呢？

### 解决方案

[官方文档](https://docs.gitlab.com/ci/docker/using_docker_images/#define-an-image-from-a-private-container-registry)在这里，
最简单的方法总结如下：

- 在本机登录一下，例如 `docker login registry.example.com:5000 --username my_username --password my_password`
- 设置CI/CD变量 `DOCKER_AUTH_CONFIG`，内容是 `~/.docker/config.json` 的文件内容

> 虽然也可以设置在runner所在机器的配置文件中，但是这样以后更改runner机器时，需要重新设置。

## Docker API version

当我安装了 `docker 29` 后，使用 `gitlab-runner 14.2.0` 时, 会提示下面的错误：

```text
Preparing the "docker" executor 00:09
ERROR: Failed to remove network for build
ERROR: Preparation failed: Error response from daemon: client version 1.25 is too old. Minimum supported API version is 1.44, please upgrade your client to a newer version (docker.go:842:0s)
```

解决方案在[这里](https://stackoverflow.com/questions/79817033/sudden-docker-error-about-client-api-version)：

`/etc/docker/daemon.json` 文件中，添加如下配置

```json
{"min-api-version": "1.32"}
```