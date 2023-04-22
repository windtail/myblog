---
title: "硬盘安装CentOS 5.5 DVD iso"
date: 2010-10-10T22:13:00+08:00
draft: false
---

先是在虚拟机中装了CentOS，感觉不是很爽，于是想装在硬盘上，但是手头又没有DVD光盘，怎么办呢。怎么硬盘安装呢。上网一顿搜，最后还真解决了。


 


最开始的时候我把DVD ISO文件拷贝到一个NTFS分区的根目录中，用grub for dos启动，倒是可以开始，但是最后要找DVD ISO镜像文件却怎么也找不到，后来网上有人说放NTFS分区里，认不出来，而且不能用grub for dos，得用原版的grub，于是我在windows下，用pq8.0格式化了一个ext3分区，但是ISO文件我怎么拷进却呢，一搜，果然也有办法，可以用ext2fs工具，在windows下挂载一个ext3分区，然后就和正常的分区一样操作了！把ISO文件拷过去之后，将ISO中的isolinux中的vmlinuz和initrd.img拷出来，放根目录下（用daemon tool或者ultraISO）。


 


这次我启动系统后，进入了grub命令行（我的grub是老毛桃winpe里的，看样子是原版的，反正最后好使了），


想想自己的ext分区是哪个硬盘的那个分区吧，我的是 hd2,4，即第3块硬盘的第5个分区（第一块是U盘，第二块是第一硬盘，第三块是第二硬盘），命令如下：


 


root (hd2,4)


kernel (hd2,4)/vmlinuz


initrd (hd2,4)/initrd.img


boot


 


开始安装后会要查找镜像文件位置，CentOS对硬盘的编号和grub是不一样的，硬盘优先级高一些，对于我来说，sda表示第一块硬盘，sdb表示 第二块硬盘，sdc表示U盘，每个硬盘又有自己的分区，从1开始计数，于是就有sda1, sda2, ..., sdb1, ...,于是我的镜像文件应该在sdb5中（不清楚的话也可以挨个试，呵呵）。接下来的就一帆风顺了！！


 


enjoy!


 


注：后来我尝试用这种方法安装Fedora 13居然不可以，原因是没法查找到镜像文件位置，不知道哪位大牛知道是怎么回事。


