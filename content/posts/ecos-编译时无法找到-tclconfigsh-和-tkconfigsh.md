---
title: "ecos 编译时无法找到 tclConfig.sh 和 tkConfig.sh"
date: 2011-03-15T21:36:00+08:00
draft: false
---

 


这是因为 tcl-devel tk-devel 一般系统中默认是不安装的，至少cent-os 5.5 和fedora 11是这样的，安装这两个包即可。


# yum install tcl-devel tk-devel


 


补记： ubuntu 10.04.2 上为了便于多个版本的tcl的存在，tcl被安装的位置不太一样，如tcl8.5


  

头文件目录：/usr/include/tcl8.5/     所以需要注意包含这个到include目录


tclConfig.sh目录：/usr/lib/tcl8.5/


库文件：/usr/lib/libtcl8.5.so   所以编译的时候必须指定 -ltcl8.5 而不是 -ltcl


 


而编译ecosconfig时，必须在configure时添加  --with-tcl-version=8.5


 


详见我的另一篇文章 Ubuntu 10.04.2上编译ecos工具


