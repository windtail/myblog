---
title: "开机校对系统时间（xp）"
date: 2011-12-25T17:54:00+08:00
draft: false
---

背景和需求
=====


家里的老机器电池坏了，老爸不会整，说查个天气预报还要把日期打全才行，不然显示的就是几年前的天气预报，问能不能整下。


于是，我就想找一个软件，双击下就校对下时间，最好不需要安装，开机也不要自启动，也不要整成系统服务，就是平时不要占系统任何资源，因为家里的机器实在是不行。


解决方案
====


在网上整了一圈，发现我这种需求还真是难满足。Windows自己的同步功能，又没有开机同步这一说，同步间隔也不能设置太小。


最后参考了这种两篇文章，编了点js的程序，搞定。  




<http://oicu.cc.blog.163.com/blog/static/12303947120103276153104/>


<http://blog.csdn.net/cnming/article/details/2200970>


JScript程序
=========



```
var oShell = new ActiveXObject("WScript.Shell");
var ret;
var i, j;
var ntpServers = ["210.72.145.44", "133.100.11.8", "203.117.180.36", "stdtime.gov.hk", 
 "130.149.17.21", "clock.via.net", "133.100.9.2", "ntp.nasa.gov"];

RETRYLOOP:
for(i=0;i<20;i++) {
	for(j=0;j=20) {
 WScript.Echo("更新时间失败，网络可能有问题！\n检查网络后，重试！");
}
```
整个解决方案，下载地址：<http://download.csdn.net/detail/windtailljj/3967829>  

  




