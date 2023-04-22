---
title: "linux find usb disk"
date: 2022-05-19T20:14:00+08:00
draft: false
---



```
lsblk -o NAME,TRAN,MOUNTPOINT | grep -A 1 -w usb | grep -v usb | awk '{print $2}'
```


https://askubuntu.com/questions/893320/what-is-the-command-for-finding-the-usb-memory-sticks-mount-point-or-path


