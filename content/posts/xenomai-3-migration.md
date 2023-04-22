---
title: "Xenomai 3 migration"
date: 2017-01-08T23:50:00+08:00
draft: false
---

Xenomai 3 的rtdm驱动更像一般的Linux驱动，named device会在/dev/rtdm/xxx创建一个设备文件。而用户空间使用时，写得来也和Linux的一般char设备相似，open/close/read/write/ioctl，只不过实际上在link的时候这些函数都被做了手脚，替换成了libcobalt.so中的函数（参见 /usr/xenomai/lib/cobalt.wrappers）。这样就导致在使用rtdm驱动时无须 xeno-config --skin=rtdm --cflags / --ldflags，而是用api对应的配置即可，例如如果用posix api，则应使用 xeno-config --posix --cflags / --ldflags


### 自动初始化


默认link时会自动加上 /usr/xenomai/lib/xenomai/bootstrap.o，并配合上相应的ldflags，直接把用户的main函数替换为xenomai\_main，读取相关的命令行参数，同时调用xenomai\_init初始化xenomai各个模块。如果一个动态库使用了rtdm，那么 xeno-config --rtdm --ldflags也会默认带上 bootstrap.o，最后link的时钟会导致 xenomai\_main重复定义无法编译过，即使编译过也会导致assert错误或者xenomai\_init被多次调用，所以库使用的ldflags应该在生成时加上--no-auto-init，即 xeno-config --rtdm --ldflags --no-auto-init


### nrt和rt的函数调用


rtdm驱动中有一些函数有两个版本，例如 ioctl\_nrt和ioctl\_rt，在Xenomai 2时会根据当前是primary还是secondary来调用rt或nrt，但是Xenomai 3不是这样了，如果有rt版本，那么会先切到primary去调用rt的版本，如果rt返回－ENOSYS（也可不实现ioctl\_rt），就会切到secondary再调用nrt的版本。这个在 [https://xenomai.org/migrating-from-xenomai-2-x-to-3-x/#Adaptive\_syscalls](https://xenomai.org/migrating-from-xenomai-2-x-to-3-x/#Adaptive_syscalls "Adaptive_syscalls") 讲了，如果不注意这一项，在primary时调用了其他linux内核函数，可能导致系统死机（不响应）


### mmap


和普通的Linux驱动一样，也支持mmap系统调用了。rtdm驱动只要实现ops->mmap即可，这个函数实现也比较简单，根据内存的性质调用 rtdm\_mmap\_vmem/rtdm\_mmap\_kmem/rtdm\_mmap\_iomem等函数即可。


 


