---
title: "Linux U盘只读解决方法"
date: 2018-01-09T16:36:00+08:00
draft: false
---

Linux Fat的U盘只读，这个问题经常出现，原因大家都说了是U盘的错误，出现这种情况后，一般的解决方案是




```
mount | grep  # 找到你的U盘的对应的设备名称，如 /dev/sdb1
sudo dosfsck -v -a /dev/sdb1
```


我今天遇到的问题，用文件管理器可以删除文件，但是往里拷贝文件时提示只读。执行以上操作后，没有任何改变。抱着试一试态度，我发现居然用命令行是可以拷贝的，再google之，解决方案是




```
killall nautilus
```


不过，我没有nautilus进程，取而代之的是 nautilus-desktop，他俩好像就差是否有桌面图标了（tweak中打开了桌面图标选项）。不管是哪个kill掉，然后再重新打开文件管理器，再往里拷贝就可以了!!


