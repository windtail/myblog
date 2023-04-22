---
title: "嵌入式（armcc）自动调用函数"
date: 2013-04-26T21:52:00+08:00
draft: false
---

有些时候，我们只想定义函数，却不想手动调用函数，而是希望这些函数在系统上电时自动调用。比如在写设备驱动时，设备的初始化函数就是这样一种函数，我们希望所有设备在上电的时候被初始化，每次增加一个设备时，不需要想着去调用这个函数，定义完之后，自己就被加入到设备初始化函数集中。


解决的方案有很多，比如写个预处理脚本，把特定格式声明的函数扫描上来，生成一个数组，然后统一调用，不过，这里有另一种解决方法：为每一个函数定义一个指针变量，然后将这个指针变量放到特定的section中，链接器最终会把同名的section组合到一起，即自动生成一个函数指针数组，访问链接器生成的符号即可。




```
typedef void(*DeviceInitFunction_t)(void);

#define DEVICE_INIT_FUNCTION(func) \
  DeviceInitFunction_t func##Ptr __attribute__((section("sectionName"))) = func

// 定义一个设备初始化函数
DEVICE_INIT_FUNCTION(InitI2c);
void InitI2c(void) {
  // ...
}

// 统一调用所有的设备初始化函数
void InitAllDevices(void) {
  extern int sectionName$$Base;
  extern int sectionName$$Length;

  DeviceInitFunction_t *initFunc = (DeviceInitFunction_t *)&sectionName$$Base;
  size_t count = （（size_t)(&sectionName$$Length))/sizeof(DeviceInitFunction_t);
  while(count--) {
    (*initFunc)();
    initFunc++;
  }
}

```


 


