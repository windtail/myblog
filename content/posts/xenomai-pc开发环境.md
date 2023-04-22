---
title: "Xenomai PC开发环境"
date: 2018-01-22T11:48:00+08:00
draft: false
---

这两天总在纠结编译一个PC机上的Xenomai开发环境，选择编译器、kernel版本和IPIP版本，但是今天忽然想到，上位机只是个开发环境，只要能编译、能运行就可以了，实时性根本不是关注的东西。而Xenomai 3刚好可以直接在PREEPT\_RT上运行，Debian自带的内核源代码里就有PREEPT\_RT，这下不用担心内核版本和各种库和程序的兼容性的问题了！


而且貌似有些 PREEPT\_RT 大有要[获胜的趋势](http://blog.machinekit.io/2015/11/and-winner-is-rt-preempt.html)，我相信这也是为什么 Xenomai 3要做成可以在 PREEPT\_RT 上运行的原因，如果有一天 PREEPT\_RT 性能足够好，用户就可以简单的迁移到 PREEPT\_RT 上了，Xenomai 3的维护人员就解放了！


编译mercury核比较简单的，内核可以不做任何修改，编译一个用户空间库就可以了




```
make -p $xeno-build && cd $xeno-build
$xeno-root/configure --with-core=mercury --enable-smp --enable-pshared
make
sudo make install
```


有了上面的库，搞搞用户空间的任务，学习下就没什么问题。不过要搞rtdm驱动之类的，还是要cobalt核


