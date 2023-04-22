---
title: "ubuntu show grub"
date: 2022-06-30T17:52:00+08:00
draft: false
---

ubuntu默认不显示grub界面，可是机器启动不了了，[这里](https://wiki.ubuntu.com/RecoveryMode)有说明。简单说就是，如果开机时按ESC（如果不好使，下次再尝试Shift）

然后grub有了之后，启动命令行又不显示。这个要改grub的菜单项中，linux启动参数，把 quiet 和 splash 去掉

启动了之后，赶紧打开 `/etc/default/grub`，注释掉`GRUB_TIMEOUT_STYLE`，修改`GRUB_TIMEOUT`为想要的值

```
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=4
```

