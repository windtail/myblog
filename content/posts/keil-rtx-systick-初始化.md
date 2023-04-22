---
title: "Keil RTX systick 初始化"
date: 2013-06-16T16:02:00+08:00
draft: false
---

在STM32F215上移植Keil的RTX操作系统，随便设置下就能好使，但是当我想知道systick到底是怎么设置的时候，就得翻翻代码了，原来在 rt\_HAL\_CM.h中以一个内联函数的形式定义的




```
1 __inline void rt_systick_init (void) {
2   NVIC_ST_RELOAD  = os\_trv;
3   NVIC_ST_CURRENT = 0;
4   NVIC_ST_CTRL    = 0x0007;
5   NVIC_SYS_PRI3  |= 0xFF000000;
6 }
```


注意：CLKSOURCE位被写死为内核时钟（FCLK），比较鄙视这种写死的方法，如果要改还得重新编译RTX的库。这里还需要说明下FCLK的频率究竟是多少的问题。简单的说，FCLK和HCLK的频率是相同的，FCLK和HCLK不同的是HCLK即使停了（休眠），FCLK仍然在运行。关于频率相同这一点可以参考STM32库中的misc.c文件中的SysTick\_CLKSourceConfig()函数，CLKSOURCE置1的时候是SysTick\_CLKSource\_HCLK


如果不想改这个文件的话，就只能改 OS\_CLOCK 和 OS\_TICK这两个宏定义了，最终这两个宏定义的乘积/1E6（OS\_TRV，参见RTX\_Conf\_CM.c文件）会被赋给os\_trv常量（参见RTX\_lib.c文件）


另注：芯片手册里说明了systick的补偿值固定为15000，是指在CLKSOURCE清0时（HCLK/8）时1ms的时间（不是CM3手册里的10ms时间，鄙视ST不按套路来）


