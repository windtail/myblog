---
title: "Cent-OS 静态IP配置"
date: 2011-04-07T21:44:00+08:00
draft: false
---

 


以前总是用NetworkManager服务，这次也不知道为什么，还是我自己不小心修改了什么，在CentOS 5.5下面用NetworkManager就是不好使，虽然能够正确地获得所有IP、mask、gateway、dns，但是上网总有问题，当上网上不去的时候，ping网关就ping不通，ping了一会儿之后，又通了，这时就能够上网了。总出现这种情况，实在是让我感觉很是不爽，于是干脆用静态IP算了。


 


 


经过一番搜索，静态配置主要包含几个配置文件：


 


/etc/sysconfig/network


--------------------内容如下------------------------


NETWORKING=yes  

HOSTNAME=xxxx


----------------------------------------------------


 


/etc/sysconfig/network-scripts/ifcfg-eth0 


--------------------内容如下------------------------


# Realtek Semiconductor Co., Ltd. RTL8111/8168B PCI Express Gigabit Ethernet controller  

DEVICE=eth0  

BOOTPROTO=static          #static 静态； dhcp 动态；none 手动  

HWADDR=90:E6:BA:34:EA:60  

ONBOOT=yes  

TYPE=Ethernet  

USERCTL=yes  

IPV6INIT=no  

PEERDNS=yes  

IPV6\_AUTOCONF=no  

IPADDR=192.168.1.221  

NETMASK=255.255.255.0  

GATEWAY=192.168.1.254  

BROADCAST=192.168.1.255  

DNS1=202.102.134.68  

DNS2=202.106.0.20


------------------------------------------------------------


 


/etc/resolv.conf


dns的信息据说也可以写在这儿，不过我还是直接写到ifcfg-eth0里了


这个文件的格式是：


nameserver 


search domain


domain domain


 


------------------------------------------------------------


后记：不知道为什么这么设置之网上网依旧比相同机器用 Windows XP 要慢一些，真是有点搞不懂了。


之前的问题也没有解决。有木有高手啊！？


