---
title: "sudo保持环境变量"
date: 2018-04-04T17:36:00+08:00
draft: false
---

编译Linux内核的最后是make modules\_install install，这两个一般都需要root权限，即sudo，而一般我交叉编译内核时都是在.bashrc中export ARCH=arm CROSS\_COMPILE=arm-linux-gnueabihf- 等等，而sudo默认会复位掉环境变量，导致设置的变量无效。




```
sudo visudo
```


执行以上命令，在打在的文件中 Defaults env\_reset行下面加一行




```
Defaults env_keep="ARCH CROSSED\_COMPILE DESTDIR INSTALL\_MOD\_PATH"
```


再执行sudo make modules\_install就可以了


 


参考文献: <https://www.chenyudong.com/archives/sudo-keep-env.html>


