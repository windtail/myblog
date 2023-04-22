---
title: "ubuntu 10.04 xdmcp"
date: 2011-07-18T23:21:00+08:00
draft: false
---

  


WinXP远程登录Ubuntu主机

  


######################################

Ubuntu主机  



```
第一步：编辑 /etc/gdm/custom.conf
```

```

```
[daemon]
User=
Group= (maybe the same as your name) 

[security]

[xdmcp]
Enable=true
DisplaysPerHost=2
HonorIndirect=false
MaxPending=4
MaxSessions=16
MaxWait=30
MaxWaitIndirect=30
PingIntervalSeconds=60
Port=177

[greeter]

[chooser]
Multicast=false

[debug]
Enable=false
```
  

```
##########################################  


Windows主机  


第二步： 下载Xmanager 3.0 企业版并安装（这个比XWin32好用多了）  


第三步：运行 Xmanager-Broadcast 可找到Ubuntu主机，连接之即可

转发注册码：081129-116663-999782  


  


参考： http://blog.csdn.net/liukeforever/article/details/5828227

  


后记：昨天还好使，今天就登不上了，真是不知道为什么。用wireshark抓包显示XDMCP Accept，

然后就是从主机来的一堆SYNC包，再过一会就弹出连接失败，咋回事呢？？！！

  


  


  



