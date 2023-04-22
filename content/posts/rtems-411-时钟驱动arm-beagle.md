---
title: "rtems 4.11 时钟驱动（arm, beagle）"
date: 2016-07-29T19:05:00+08:00
draft: false
---

根据bsp\_howto手册，时钟驱动的框架主要在 **c/src/lib/libbsp/shared/Clockdrv\_shell.h** 文件中实现


 时钟初始化
------


时钟驱动初始化函数为 Clock\_initialize()，这个函数在哪里被调用了呢？


**cpukit/include/rtems/clockdrv.h** 中定义了 CLOCK\_DRIVER\_TABLE\_ENTRY




```
#define CLOCK_DRIVER_TABLE_ENTRY \
 { Clock\_initialize, NULL, NULL, NULL, NULL, NULL } 
```


 


然后将 CLOCK\_DRIVER\_TABLE\_ENTRY 放到了IO驱动列表中，在初始化的时候应该就会被调用了。


  
Clock\_initialize() 函数定义在 c/src/lib/libbsp/shared/Clockdrv\_shell.h 中，调用了BSP提供的2个函数：   
Clock\_driver\_support\_install\_isr() => 安装中断处理函数 Clock\_isr  
Clock\_driver\_support\_initialize\_hardware() => 初始化硬件   
   
再看Clock\_isr()，在中断处理的结束位置调用了 Clock\_driver\_support\_at\_tick() ，这个函数可以用来通知硬件中断响应已经完成。  
c\_user手册中说明的 Clock\_isr() 应该调用的 rtems\_clock\_tick() 函数，这是在 Clock\_isr()=>Clock\_driver\_timecounter\_tick()中完成的， 不过，现在调用更新版本的rtems\_timecounter\_tick()，更现代的时钟驱动需要一个精准的时钟，每个tick中断时， 去读这个精准的时钟，这样就可以提高定时的精度了。   
   
Clock\_exit() 函数中调用的 Clock\_driver\_support\_shutdown\_hardware() 也是BSP可以提供的函数，用于停止时钟运行。  
 


### beagle


再看beagle BSP的代码，在 **c/src/lib/libbsp/arm/beagle/Clock.c** 中，直接到结束位置看就行了  
 




```
#define Clock_driver_support_at_tick() beagle_clock_at_tick()
#define Clock_driver_support_initialize_hardware() beagle_clock_initialize()
#define Clock_driver_support_install_isr(isr, old_isr) \
  do { \
 beagle\_clock\_handler\_install(isr); \
 old\_isr = NULL; \
 } while (0)
 
#define Clock_driver_support_shutdown_hardware() beagle_clock_cleanup()
 
/\* Include shared source clock driver code \*/
#include "../../shared/clockdrv\_shell.h" 
```


 beagle\_clock\_initialize() 中使用了两个时钟，一个不停的跑，用作精准时钟（以最高频率运行），另一个仅用于产生tick中断。注意beagle\_clock\_initialize()函数中调用了rtems\_configuration\_get\_microseconds\_per\_tick()，这个是从配置表中获取，由应用程序编写人员通过宏定义配置tick时间。


