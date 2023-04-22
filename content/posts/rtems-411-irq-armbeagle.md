---
title: "rtems 4.11 IRQ （arm,beagle）"
date: 2016-08-03T20:17:00+08:00
draft: false
---

arm IRQ入口在 cpukit/score/arm/arm\_exec\_interrupt.S 中，其中BSP最关心就是 **bl bsp\_interrupt\_dispatch** 这句，看看beagle BSP的实现， c/src/lib/libbsp/arm/beagle/irq.c，实现很简单，找到是哪一个中断源（vector number）引起的中断，然后调用 bsp\_interrupt\_handler\_dispatch 即可，最后中断处理完后，通知中断控制器中断处理结束，可以引入下一个中断了。


中断相关的几个函数：


* bsp\_interrupt\_facility\_initialize()：中断控制器初始化
* bsp\_interrupt\_vector\_enable()：使能中断控制器产生中断
* bsp\_interrupt\_vector\_disable()：禁止中断控制器产生中断
* bsp\_interrupt\_dispatch()：找到中断源，然后调用bsp\_interrupt\_handler\_dispatch

