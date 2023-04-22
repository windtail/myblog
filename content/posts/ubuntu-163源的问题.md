---
title: "Ubuntu 163源的问题"
date: 2011-04-20T23:02:00+08:00
draft: false
---

 


W: GPG签名验证错误： http://ppa.launchpad.net
 karmic Release: 由于没有公钥，下列签名无法进行验证： NO\_PUBKEY FA9C98D5DDA4DB69的解决办法  

  

出现以上错误提示时，只要把后八位拷贝一下来，并在[终端]里输入以下命令并加上这八位数字回车即可！  

sudo apt-key adv --recv-keys --keyserver keyserver.Ubuntu.com DDA4DB69  

  

此类问题均可如此解决！


 


转自：<http://forum.ubuntu.org.cn/viewtopic.php?f=74&t=269519&p=1855333>


