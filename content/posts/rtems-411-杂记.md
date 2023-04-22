---
title: "rtems 4.11 杂记"
date: 2016-08-07T21:01:00+08:00
draft: false
---

object id
---------


task, message queue, semaphore, memory region, memory partition, timer, port, rate monotonic period 都是对象，对象引用时都用ID，32位的ID定义如下




|  |  |
| --- | --- |
| 位 | 作用 |
| 0-15 | object index |
| 16-23 | node (cpu) |
| 24-26 | API |
| 27-31 | class |


其中每一个字段都可以通过 rtems\_object\_id\_get\_api/class/node/index()获取，这样定义的object id可以快速查找到object


rtems\_build\_name() 可以取一个4字节的名称给object，一般用4个字符，这样方便调试时辨别。


任务
--


RTEMS 支持 255 个任务优先级，其中 255 是最低优先级，而 1 是最高优先级，注意没有0级。


任务有5种状态： executing, ready, blocked, dormant（已创建但还未start的任务）, non-existent


### 浮点运算


浮点运算，任务需要浮点运算时，应在创建时指定 RTEMS\_FLOATING\_POINT 属性


初始化
---


初始化用的栈在初始化之后，RTEMS不再使用，可用作中断处理


为了使所有任务同时开始执行，我们可以设置初始化任务不可抢占，当初始化任务删除自己时，所有任务开始执行


POSIX
-----


RTEMS 不支持 内存保护（memory protection），不能实现多进程，所以只实现了单进程，多线程的模型，整个应用被作为一个进程（任务应该就是线程了，为什么不把任务视为进程呢）


