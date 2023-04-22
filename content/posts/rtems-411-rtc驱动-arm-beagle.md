---
title: "rtems 4.11 RTC驱动 （arm, beagle）"
date: 2016-08-03T20:48:00+08:00
draft: false
---

RTC驱动的框架在 c/src/lib/libbsp/shared/tod.c 中，大部分功能都已经实现了，入口函数是 rtc\_initialize()，BSP要实现的东西非常少。


beagle的实现在 c/src/lib/libbsp/arm/beagle/rtc.c中，提供一个 rtc\_tbl RTC\_Table[] 数组，数组的大小存储在 RTC\_Count 全局变量中，每一个RTC\_Table元素就是一个可能的RTC芯片，rtc\_initialize()时，会调用每个RTC\_Table元素的probe函数，第一个返回true的元素就是系统的rtc设备，这种实现方式是为了方便兼容产品的不同型号的主板（例如RTC芯片停产，换了另一个RTC芯片）。


RTC\_Table元素（RTC设备）
-------------------


RTC设备必须 rtc\_fns 结构体中的3个函数：


* 初始化：打开RTC设备时钟，设置总线访问方式等等
* 读：从RTC设备中读取时间
* 写：把时间设备到RTC设备中


rtc\_initialize调用
-----------------


cpukit/libcsupport/include/rtc.h 中定义了 RTC\_DRIVER\_TABLE\_ENTRY，如果最终应用需要rtc，则可以通过配置，在 cpukit/sapi/include/confdefs.h 中包含到 \_IO\_Driver\_address\_table 中去


