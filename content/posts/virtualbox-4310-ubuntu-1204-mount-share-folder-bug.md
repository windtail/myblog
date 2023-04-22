---
title: "virtualbox 4.3.10 ubuntu 12.04 mount share folder bug"
date: 2014-04-30T00:21:00+08:00
draft: false
---

virtualbox 4.3.10 不能mount共享文件夹，这是一个bug，参考如下链接


<https://www.virtualbox.org/ticket/12879>


执行以下命令即可：  
sudo ln -s /opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions


