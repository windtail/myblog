---
title: "网络启动并安装Debian"
date: 2018-02-01T17:25:00+08:00
draft: false
---

网络启动（PXEBoot）并安装Debian的官方文档在[这里](https://wiki.debian.org/PXEBootInstall)，不过官方文档有点冗长，我这里假设已经有一台安装好Debian，需要网络安装另一台（这台可以是虚拟机，通过ISO文件等等方式安装的）。PXE需要两个服务，tftp和dhcp，不过debian中dnsmasq一个软件包全部搞定




```
sudo apt-get install debian-installer-9-netboot-amd64 dnsmasq  
**（注意：9表示是debian stretch，amd64是架构）**
```


打开 /etc/dnsmasq.conf，搜索并打开（uncomment）以几个注释行




```
dhcp-range=192.168.7.50,192.168.7.55,255.255.255.0,12h
dhcp-boot=pxelinux.0
pxe-service=x86PC, "Install Linux", pxelinux
enable-tftp
tftp-root=/usr/lib/debian-installer/images/9/amd64/gtk
```


我只是修改了dhcp-range和tftp-root两行，其他都是直接打开注释的，dhcp-range这一行要注意前两个IP必须和你本机是同一个网络内的，当然不能和网内其他主机冲突了。


把目标机器设置为从网络启动，然后就可以正常开始安装了。这只是开始安装而已，因为这样配置的目标机器实际是不能上网（连接internet），所以还需要用apt-mirror把debian镜像下来，然后在本机安装nginx，让目标机器使用本地mirror。当然我们也可以配置DHCP服务器，使目标机器可以连接internet，那就请参照官方文档了。


