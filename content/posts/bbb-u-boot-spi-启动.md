---
title: "bbb u-boot SPI 启动"
date: 2018-03-26T09:36:00+08:00
draft: false
---

beagle bone black的u-boot编译时已经为SPI准备好了 MLO.byteswap，这个文件应该直接写入到SPI flash的偏移0位置，根据am335x的手册，SPI内可以保存多份引导，具体参见手册。




```
U-Boot# sf probe 0
U-Boot# sf erase 0 +E0000
U-Boot# mmc rescan
U-Boot# fatload mmc 0 ${loadaddr} MLO.byteswap
U-Boot# sf write ${loadaddr} 0 ${filesize}
U-Boot# fatload mmc 0 ${loadaddr} u-boot.img
U-Boot# sf write ${loadaddr} 0x80000 ${filesize}
```


以上是TI官方给出的如何把MMC引导的u-boot写入到SPI的命令，这些命令假设了 MLO.byteswap和u-boot.img在mmc的第一分区中，并且还假设了MLO（即u-boot-spl）会从0x80000位置读u-boot，实际我们自己的板子可能不是0x80000，翻了翻u-boot的代码，在spl\_spi.c文件中发现了它，在spl\_spi\_load\_image函数中，从CONFIG\_SYS\_SPI\_U\_BOOT\_OFFS偏移位置加载了u-boot，CONFIG\_SYS\_SPI\_U\_BOOT\_OFFS一般定义在 include/configs/.h中  
  



