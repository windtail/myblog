---
title: "STM32 I2C"
date: 2013-06-19T22:45:00+08:00
draft: false
---

STM32 I2C 搞了几天了，比较郁闷，写点东西给那些正在郁闷的同志




```
// 好使的，也是范例的代码
cnt = TIME\_OUT;
while (cnt-- && !I2C\_ChechEvent(I2C2, XXX));
if (!cnt) goto err;

// 不好使，总是超时
cnt = TIME\_OUT;
while (!I2C\_ChechEvent(I2C2, XXX)) {
 cnt--;
 if (cnt == 0) goto err;
}

// 死在这了，动不了
while (!I2C_ChechEvent(I2C2, XXX));
```


另一个问题，如果只初始化I2C2，我的I2C外设芯片经常不能正常工作，后来发现如果这个时候把I2C3也使能，然后差不多配置下，  
居然就好使了。**别问我为什么**，郁闷的同志可以考虑按照这个方法试试（急病乱投医啊！！！）


