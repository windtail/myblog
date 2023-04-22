---
title: "rtems 4.11 启动流程（arm, beagle）"
date: 2016-07-29T18:24:00+08:00
draft: false
---

请参照官方的 bsp\_howto 文档，对arm来说，首先执行的文件是start.S


start.S
-------


**c/src/lib/libbsp/arm/shared/start/start.S**


1、从 \_start 开始执行   
2、关CPU中断，初始化寄存器，设置好各mode的栈，调用 bsp\_start\_hook\_0()，注意：此时异常向量表还没有配置好   
3、然后拷贝vector到真正的位置（但没有设置cp15）   
4、调用 bsp\_start\_hook\_1()   
5、最后调用 boot\_card()，这个函数不应该返回，如果不幸返回，将会调用 bsp\_reset() 重启   
   
根据 bsp\_howto 文档，在调用boot\_card()之前应该做的事情如下：   
1、init stack   
2、zero .bss   
3、disable irq   
4、copy .data to RAM   
其中第2和4项没有在start.S中出现，它们在哪里呢？ 让我们看看beagle的bsp\_start\_hook\_0()和bsp\_start\_hook\_1()，它们在  
   
**c/src/lib/libbsp/arm/beagle/startup/bspstarthooks.c**   
   
**bsp\_start\_hook\_0**() 什么也没干   
   
**bsp\_start\_hook\_1**()   
  arm\_a8core\_start\_hook\_1() => **c/src/lib/libbsp/arm/include/arm-a8core-start.h**  告诉cp15，中断向量的位置   
  bsp\_start\_copy\_sections() => **c/src/lib/libbsp/arm/include/start.h**  拷贝标准sections到内存，包括.data初始化数据到内存   
  beagle\_setup\_mmu\_and\_cache() => **c/src/lib/libbsp/arm/beagle/startup/bspstartmmu.c** 设置板子的mmu   
  bsp\_start\_clear\_bss() => **c/src/lib/libbsp/arm/include/start.h**  清零.bss段   
   
至此 bsp\_howto 中boot\_card()之前的各项任务都已经完成，接下来看看 boot\_card() 


boot\_card()
------------


boot\_card()的默认实现在 **c/src/lib/libbsp/shared/bootcard.c**，实现很简单：   
1、关中断   
2、记录命令行（如果bootloader是uboot，可能会有命令行）   
3、调用 rtems\_initialize\_executive() 进入rtems的核心地带~   
   
rtems\_initialize\_executive()在 **cpukit/sapi/src/exinit.c**   
把所有注册的 rtems\_sysinit\_item （初始化项）都调用一遍，之后操作系统正式开始运行   
   
话说 rtems\_sysinit\_item 是哪里来的呢？说到这里咱还得回想下 bsp\_howto 手册里说好的 bsp\_work\_area\_initialize/bsp\_start/bsp\_predriver\_hook都哪里了，回头看看 bootcard.c 吧，在boot\_card()函数之上还有这些定义




```
RTEMS\_SYSINIT\_ITEM(
 bsp\_work\_area\_initialize,
 RTEMS\_SYSINIT\_BSP\_WORK\_AREAS,
 RTEMS\_SYSINIT\_ORDER\_MIDDLE
);
 
RTEMS\_SYSINIT\_ITEM(
 bsp\_start,
 RTEMS\_SYSINIT\_BSP\_START,
 RTEMS\_SYSINIT\_ORDER\_MIDDLE
);
 
RTEMS\_SYSINIT\_ITEM(
 bsp\_predriver\_hook,
 RTEMS\_SYSINIT\_BSP\_PRE\_DRIVERS,
 RTEMS\_SYSINIT\_ORDER\_MIDDLE
); 
```


rtems\_sysinit\_item 就是这么定义的！那它们的顺序是怎么样的呢，看看 **cpukit/score/include/rtems/sysinit.h** 大概就是按照这个定义的顺序，至于如何按定义的顺序执行，我现在也不知道，先挖个坑 。  
   
bsp\_work\_area\_initialize() 在 **c/src/lib/libbsp/shared/bspgetworkarea.c**，一般采用默认实现   
bsp\_start() 在 **c/src/lib/libbsp/arm/beagle/startup/bspstart.c** 调用 bsp\_interrupt\_initialize() => bsp\_interrupt\_facility\_initialize() 初始化中断控制器（正如bsp\_howto手册里说的一样）（在 **c/src/lib/libbsp/arm/beagle/irq.c** 中）   
bsp\_predriver\_hook() 默认实现在 **c/src/lib/libbsp/shared/bsppredriverhook.c** 什么也没做 


其他
--


bsp\_reset()在 **c/src/lib/libbsp/arm/beagle/startup/bspreset.c** 主要作用就是重启系统   
   
beagle\_setup\_mmu\_and\_cache()初始化MMU，根据section配置表初始化MMU，而这个表大部分都是 ARMV7\_CP15\_START\_DEFAULT\_SECTIONS，定义在 **c/src/lib/libbsp/arm/shared/include/arm-cp15-start.h** 中！   
   
至此，rtems操作系统的初始化步骤已经大体清晰了^\_^


