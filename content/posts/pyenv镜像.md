---
title: "pyenv镜像"
date: 2021-03-13T21:10:00+08:00
draft: false
---

在linux下使用新的python真是一件不容易的事情，之前总是python源代码下载不下来，还把pyenv的源代码下载部分都加上了proxychains，今天又看了下源代码，其实人家早就想到了。




```
export PYTHON_BUILD_MIRROR_URL_SKIP_CHECKSUM=1 
export PYTHON\_BUILD\_MIRROR\_URL=https://npm.taobao.org/mirrors/python
```


在.bashrc中加上以上两句就OK了


