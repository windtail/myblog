---
title: "Xenomai 3 POSIX"
date: 2018-01-24T10:44:00+08:00
draft: false
---

Xenomai 3在架构设计上确实优先Xenomai 2，至少对开发者来说，少维护了不少东西，看下面两张图就知道了


![xenomai 2 architecture](http://xenomai.org/wp-content/uploads/2016/10/x2-interfaces.png "Xenomai 2的架构图")


![xenomai 3 architecture](http://xenomai.org/wp-content/uploads/2016/10/x3-cobalt-interfaces-3.png "Xenomai 3的架构图")


第一张图是Xenomai2的，第二张图是Xenomai3的，Xenomai3在内核中只有一个cobalt core，并没有POSIX/native/VxWorks等等的封装，内核的代码本来就不易于调试，也就不易于维护（保持正确性），减少内核代码就有利于代码的稳定性。最重要的是只有一个cobalt，大大减轻了维护人员的工作。


注意，第二张图有一个重要的提示，**POSIX在Xenomai3中的地位非一般**，以前大家习惯了native的API，Xenomai3时native的API比POSIX的API要慢一点了，因为native是在POSIX上封装出来的，而不是直接给内核打交道。


另外，POSIX API的好处时，如果PREEPT\_RT的真的能达到硬实时性的话，Xenomai用户空间的库就不需要了，看看mercury core的图就知道了


![xenomai 3 architecture (preept_rt)](http://xenomai.org/wp-content/uploads/2016/10/x3-mercury-interfaces-3.png "Xenomai 3的架构图（mercury）")


POSIX API直接被glibc支持，Xenomai3的用户空间的库都不需要了。


总体看来，我认为是时候切换到Xenomai3的了，虽然Xenomai3目前并没有Xenomai2稳定，但是架构放在这里，稳定只是时间问题。一旦稳定下来，维护人员就可以花时间去优化实时性了。


