---
title: "is char signed or unsigned?"
date: 2018-11-30T17:26:00+08:00
draft: false
---

工作这么多年，一直认为char是有符号的，而事实上gcc和vs默认也是有符号，但是c规范里实际并没有指明char是有符号还是无符号，所以char比较特殊，


* char
* signed char
* unsigned char


是三种数据类型，与int等是不一样的，int就等效为signed int。


gcc和vs都有选项设置char为unsigned。


**arm开发程序员需要注意 armcc和armclang的char都是无符号的**。


