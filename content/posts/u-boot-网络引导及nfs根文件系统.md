---
title: "u-boot 网络引导及NFS根文件系统"
date: 2018-03-26T09:50:00+08:00
draft: false
---

u-boot可以从tftp服务器下载内核及dtb文件，然后通过bootargs来传入内核需要的根文件系统参数。


TFTP服务器
-------




```
sudo apt install tftpd-hpa  # 注意是tftpd不是tftp，没有d的是客户端
```


默认tftp服务就启动了，默认的根目录是 /srv/tftp，为了避免权限问题，一般把 /srv/tftp改为软连接到HOME的某个子目录。另外tftp监控的端口是69，但是如果在防火墙后面（如虚拟机），也不太好配置，因为tftp的数据端口会使用任意端口，据说port-range可以设置数据端口使用的范围，不过，我现在直接在Linux下工作，不用虚拟机，如果在Windows中，建议直接使用Windows版本的tftp服务器。


注：虚拟机也可以配置的桥接网卡的方式，不过，那样虚拟机不能通过主机上网了。


 


 


内核配置


CONFIG\_ROOTFS\_NFS


 


nfs-kernel-server


/etc/exports


exportfs -a


本地mount尝试


静态ip ${ipaddr}:::::eth0


 


无法mount


抓包发现只使用的v2协议，然后没有再尝试其他协议，内核配置里也使能了V3/V4协议，没发现默认使用什么协议mount


增加nfs选择 nfsvers=3，即可


 


