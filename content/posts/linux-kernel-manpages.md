---
title: "Linux kernel manpages"
date: 2018-01-25T09:56:00+08:00
draft: false
---

<https://www.linuxquestions.org/questions/linux-newbie-8/man-pages-for-kernel-functions-758389/>


在Linux内核中执行以下两条命令即可




```
make mandocs
sudo make installmandocs
```


生的manpages在 /usr/local/man/man9中，不是所有的函数都有，可以ls下这个目录，看看有哪些




```
man netdev_alloc_skb
```


至少有一些函数不用的翻代码了


