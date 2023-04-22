---
title: "Rate Monotonic Scheduling algorithm"
date: 2016-08-09T06:36:00+08:00
draft: false
---

这篇文章写得不错 <http://barrgroup.com/embedded-systems/How-To/RMA-Rate-Monotonic-Algorithm>


另外rtems的官方文档也有类似说明 <https://docs.rtems.org/doc-current/share/rtems/html/c_user/Rate-Monotonic-Manager-First-Deadline-Rule.html>


总结以下几点：


* RMS 是一个优化的静态优先级硬实时调度算法，如果能被其他静态优先级调度算法调度，那么一定可以用RMS调度
* 所有有硬实时需求的任务都是周期性的，周期越小的优先级应设置为越高
* 每个任务的CPU占用率F＝执行时间/周期，RMS能调度开的要求是 sum(F) <= n \* (2\*\*(1/n) - 1)，其中n为硬实时任务数

