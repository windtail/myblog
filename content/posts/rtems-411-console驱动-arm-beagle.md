---
title: "rtems 4.11 console驱动 （arm, beagle）"
date: 2016-08-03T20:35:00+08:00
draft: false
---

console驱动框架主要文件是 c/src/lib/libbsp/shared/console.c，驱动的入口是 **console\_initialize**()主要作用是初始化BSP提供的全局变量 **Console\_Configuration\_Ports**[Console\_Configuration\_Count]，初始化termios架构，注册console设备的文件节点。


c/src/lib/libbsp/arm/beagle/console/console-config.c 是beagle BSP提供的有关console驱动的唯一文件，由于am335x提供的串口设备与ns16550寄存器兼容，事情就变得非常简单了，因为ns16550的各种操作函数在libchip中都实现了，具体一的文件是 c/src/libchip/serial/ns16550.c，应该说很多CPU提供的串口都是ns16550兼容的，这个是事实上的标准，即使不幸不兼容，咱也不过是要实现几个操作函数而已～


C库支持
----


要支持C库的printf和scanf，必须提供输入输出函数，应初始化 BSP\_output\_char 和 BSP\_poll\_char 两个函数指针，beagle BSP提供的函数使用了和console相同的串口，并且固定为轮循方式获取或写入字符，感觉可能会出错，以后再研究。


console\_initialize调用
---------------------


cpukit/include/rtems/console.h 中定义了 CONSOLE\_DRIVER\_TABLE\_ENTRY，如果最终应用需要console，则可以通过配置，在 cpukit/sapi/include/confdefs.h 中包含到 \_IO\_Driver\_address\_table 中去


