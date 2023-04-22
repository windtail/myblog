---
title: "rtems 4.11 部分m4文件分析"
date: 2016-08-07T20:18:00+08:00
draft: false
---

本来想把configure.ac和各种m4文件分析明白，发现有点困难，不过好在也能理解一些。


基本教程
----


首先要明白m4，参见这个[教程](https://segmentfault.com/a/1190000004104696)，写得不错，不论怎么样m4替换来替换去的，还真是不那么容易懂，好在我们自己不用写，只要看懂大意而已～


然后我们还得去了解下 autoconf和automake，这本书不错 《Autotools: A Practioner's Guide to GNU Autoconf, Automake, and Libtool》，csdn有下载。


文件夹结构
-----


当autoconf和automake时使用时，用户自定义宏应该放在 acinclude.m4和m4/\*.m4（GNU autoconf手册上说好像是 aclocal/\*.m4），流程是这样的：


acinclude.m4 \  



aclocal/\*.m4 ｜ === aclocal perl script （由automake提供） ===> aclocal.m4


configure.ac /


然后 autoconf 会在处理 configure.ac 之前先处理 aclocal.m4，使 acinclude.m4 aclocal/\*.m4中包含的宏定义可在configure.ac中使用


aclocal/\*.m4
-------------


* version.m4：定义\_RTEMS\_VERSION和\_RTEMS\_API常数
* rtems-top.m4：定义RTEMS\_TOPdir＝源代码顶层目录（相对路径）和PROJECT\_TOPdir＝build-tree顶层目录（相对路径）
* enable-cxx.m4：定义 --enable-cxx 选项，最终对应变量为 RTEMS\_HAS\_CPLUSPLUS，默认为 no
* enable-drvmgr.m4：定义 --enable-drvmgr 选项，最终对应变量为 RTEMS\_DRVMGR\_STARTUP，默认为 yes
* enable-multiprocessing.m4：定义 --enable-multiprocessing 选项，最终对应变量为 enable\_multiprocessing，默认为 no
* enable-networking.m4：定义 --enable-networking 选项，最终对应变量为 RTEMS\_HAS\_NETWORKING，默认为 yes
* enable-paravirt.m4：定义 --enable-paravirt 选项，最终对应变量为 RTEMS\_HAS\_PARAVIRT，默认为 no
* enable-posix.m4：定义 --enable-posix 选项，最终对应变量为 RTEMS\_HAS\_POSIX\_API， 默认为 yes
* enable-rtemsbsp.m4：定义 --enable-rtemsbsp 选项，最终对应变量为 enable\_rtemsbsp，这是一个列表，每个--enable-rtemsbsp都会向这个列表中添加新的bsp，也可以一次性加入多个以空格分隔的bsp列表，默认为空
* enable-rtems-debug.m4：定义 --enable-rtems-debug 选项，最终对应变量为 enable\_rtems\_debug， 默认为 no
* enable-smp.m4：定义 --enable-smp 选项，最终对应变量为 RTEMS\_HAS\_SMP， 默认为 no
* enable-tests.m4：定义 --enable-tests 选项，最终对应变量为 enable\_tests， 取值可为 yes/no/samples 默认为 samples
* multilib.m4：定义 --enable-multilib 选项，最终对应变量为 multilib， 默认为 no，同时定义了automake条件 MULTILIB，将会定义两个autoconf变量，@@MULTILIB\_TRUE@@和@@MULTILIB\_FALSE@@可在makefile中使用，一个将被定义为空，另一个为#（makefile注释）
* path-ksh.m4：将 bash/ksh/sh中第一个找到的shell的绝对路径设置到KSH变量中，AC\_PATH\_PROGS和AC\_CHECK\_PROGS相似，但设置的是绝对路径
* project-root.m4：设置 PACKHEX和BIN2C变量，指向编译好的tools/build/packhex和rtems-bin2c工具
* canonical-target-name.m4：从$target变量中获取RTEMS\_CPU变量
* check-bsps.m4：定义 RTEMS\_CHECK\_BSPS 该宏将参数指定的shell变量赋于所有BSP列表（此处隐含rtems的BSP是不能重名的），所有BSP是怎么确定的呢？其实先 ls c/lib/libbsp/$RTEMS\_CPU/\*/bsp\_specs，其中 \* 就是bsp family，然后再找bsp family 文件夹下的 make/custom/\*.cfg，这就是bsp了。对于beagle来说，就是 c/lib/libbsp/arm/beagle/make/custom/\*.cfg 即 beagleboardorig / beagleboardxm / beagleboneblack / beaglebonewhite
* check-custom-bsp.m4：定义了 \_RTEMS\_CHECK\_CUSTOM\_BSP[bsp\_name, path]，查找 .cfg文件是否存在，并将全路径赋于  的shell变量
* bsp-alias.m4：定义了 \_RTEMS\_BSP\_ALIAS[bsp\_path, bsp\_family]，根据bsp配置的文件路径返回bsp\_family，bsp\_path 默认为 $RTEMS\_BSP，bsp\_family 默认为 RTEMS\_BSP\_FAMILY


acinclude.m4 文件有点复杂，没有仔细看，另外，我发现 c/src/aclocal 和 cpukit/aclocal 有很多与 顶层目录的 aclocal 目录中相似或相同的文件（这种设计应该是有问题的），暂时也用不着，先不管了，知道怎么加一个bsp就可以继续向前了。


 


