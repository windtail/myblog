---
title: "编译openocd"
date: 2016-07-22T23:36:00+08:00
draft: false
---

git clone openocd


./configure --enable-jlink 


需要libusb-1.x


下载libusb-1.0.20


./configure --disable-udev  --prefix=/usr 因为centos 7 没有libudev 估计是被systemd-udevd代替了


make


make install


sudo ldconfig


PKG\_CONFIG\_PATH=/usr/lib/pkgconfig ./configure --enable-jlink


 


