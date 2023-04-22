---
title: "Cygwin安装Gitolite3"
date: 2012-07-01T17:26:00+08:00
draft: false
---

  




Cygwin 安装Gitolite3只要注意两点就行了，别的其实没有什么问题，一切按官方的安装文档即可


1. 必须完全按官方文档，安装时必须是clone下来的git仓库（带.git文件夹）
2. 安装完后，部分功能不能使用，经常输出乱码，在 .gitolite.rc 文件的最开始添加 **$ENV{PATH} = "/usr/local/bin:/bin:/usr/bin";**


参考： <http://alone11.iteye.com/blog/1078297> （这是安装2，安装3的方式参见官方文档）  




