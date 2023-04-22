---
title: "grub备忘"
date: 2010-10-15T23:24:00+08:00
draft: false
---

 


    今天开机时，忽然发现不能启动了，直接到grub命令行。据说这是因为没有找到menu.lst文件，可是我从来没修改过啊，在命令行下，一顿尝试，又上网一顿搜，由于非常不熟悉grub命令，所以整了半天才手动把CentOS启动起来，那为什么不能自动进入系统呢。


    进入系统发现menu.lst文件中是(hd0,6)，而我手动启动时是从(hd0,5)启动的，磁盘顺序怎么会变呢。先修改menu.lst文件为(hd0,5)试试，问题依旧，这是怎么回事呢，怎么修改了没有效果！于是这认为是磁盘顺序的问题，又是一顿google，搜partition order时，发现有人求助fdisk -l有报警 Partition Table Entries not in Disk order，于是我也试了下，果然也有这个警告。按照他说的执行


  fdisk /dev/sd0  #选择硬盘1


  x                      #进入expert模式


  f                       #fix partition order


  w                     #write (apply)


  重启之后，分区的顺序果然和硬盘中的顺序相同了。但是问题是我安装的时候分区顺序就不对啊。这下，我得手动从(hd0,4)启动了。


  继续google，发现有人说修改grub.conf文件，于是打开grub.conf一看，发现和menu.lst文件之前的内容是一样的（即是(hd0,6)），崩溃。修改了这个文件中的(hd0,6)为(hd0,4)之后，重启成功启动。


  回头再google了下 grub.conf和menu.lst文件的关系，人都说grub.conf文件链接到menu.lst文件，可是在我的CentOS中，它们都是不同的文件，晕死。


 


  这次还学习了一些grub命令行，记录如下：


  (1) boot linux


      root (hd0,x)


      kernel /vmlinuz


      initrd /initrd.img


      boot


      注意：grub支持tab自动补全,和bash差不多,很方便


  (2) boot windows


      rootnoverify (hd0,0)


      chainloader +1


      boot


  (3) boot windows on second harddisk


      map (hd1) (hd0)


      map (hd0) (hd1)


      rootnoverify (hd1,0)


      chainloader +1


      boot


  (4)menu.lst 语法


       default=


       timeout=


       title xxx (start an item)


       命令boot, 在menu.lst文件中省略


  (5)grub菜单中,可按e编译命令,按c进入命令行


