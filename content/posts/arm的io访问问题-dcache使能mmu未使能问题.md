---
title: "ARM的I/O访问问题 （DCache使能、MMU未使能问题）"
date: 2011-06-10T23:03:00+08:00
draft: false
---

 


今天调试出了特别奇怪的问题，经常用的串口发送居然都不好使了，代码大约是这样的：


 


if( flag in memory mapped reg) {


  write char to transmit reg


}


 


从串口发出去的东西总是不对，用jtag调试发现，单步的时候是正确的，但是只要全速运行就出错。


 


思考之后，感觉只可能是flag读回来是不对的，但怎么可能呢？？


 


仔细思考后，发现一个问题，ARM如何知道每次都要通过读这个地址来获得flag呢？


虽然已经将memory-mapped reg 的地址定义为 volatile，但是这只是告诉gcc不优化进寄存器，但是


ARM还可以从Cache中读啊，ARM没有专用的IO指令，如何才能让它必须读内存地址呢？


 


答案是两种：


1）关闭D-Cache


2）使能D-Cache和MMU，配置MMU，使得内存映射的寄存器地址处设置为none-cached


 


而偏偏我打开了D-Cache，却未使能MMU。导致这样的结果是因为我正在做操作系统移植的最低层操作，


我想打开D-Cache可以提高速度，就打开了，但是我的操作系统不使用MMU，而我也懒得配置MMU为直接映射，


干脆就禁掉了。结果就悲催了！！也好，以后就能多注意点东西。


 


 


