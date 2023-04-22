---
title: "Debian NAT共享上网"
date: 2018-02-24T16:06:00+08:00
draft: false
---

如果Linux主机有两个网卡，比如一个有线、一个无线，当无线连接后，其他机器即可通过有线共享上网，为了方便叙述，假设环境如下：


* A机器有两块网卡，eth0和ws0，其中ws0为无线网卡，已连接wifi，而eth0为有线网卡
* B机器只有一个有线网卡，假设也为eth0


首先需要以root身份在A机器上执行以下代码（注意ws0应替换为你自己的无线网卡名称）




```
echo 1 > /proc/sys/net/ipv4/ip\_forward
iptables -t nat -I POSTROUTING -o ws0 -j MASQUERADE
```


然后需要将A、B机器的eth0设置在同一个网段内，并且A机器的eth0不能设置默认网关，假设A、B的IP分别为：


* A：192.169.0.1
* B：192.169.0.2


则我们需要把B机器的默认网关设置为A的IP，即192.169.0.1，B机器的DNS可设置为8.8.8.8，然后应该就可以上网了


 


