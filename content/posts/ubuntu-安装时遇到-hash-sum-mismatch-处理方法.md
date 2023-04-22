---
title: "ubuntu 安装时遇到 hash sum mismatch 处理方法"
date: 2016-11-26T22:11:00+08:00
draft: false
---

ubuntu安装大软件时，下载经常容易出错，hash sum mismatch是其中一种，说到底还是网络不好，重试很多遍都是这个错误，最后的解决方案是把mismatch说的那个链接用firefox打开并下载，然后 mv 到 /var/cache/apt/archives/ 下面即可


另注：在下载 ros 时，国内的镜像少了东西，报404错误，也用相似的方法解决，去国外的库里下载同路径名的文件，然后放到cahce去就可以了！


 


