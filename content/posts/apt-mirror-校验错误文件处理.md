---
title: "apt-mirror 校验错误文件处理"
date: 2018-01-27T21:43:00+08:00
draft: false
---

apt-mirror是一个用来将Debian或Ubuntu的软件源镜像到本地的工具，这个工具工作得非常好，不过有的时候由于网络问题，会有一些文件的校验是失败的，但apt-mirror并不能发现，等到最后装软件的时候发现校验错误，再去下载，太耽误时间，并且也不能一次性发现所有错误。


[apt-mirror-check](https://github.com/windtail/apt-mirror-check)工具是本人写的一个小工具，可以把校验错误的文件删除，然后再运行apt-mirror重新下载，欢迎大家试用并指正。


