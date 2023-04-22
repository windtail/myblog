---
title: "ssh不检查server变化"
date: 2018-04-04T08:35:00+08:00
draft: false
---

嵌入式linux开发时经常需要远程登录到板上，但由于开发过程还经常会重新下载内核和文件系统，导致登录时总提示host变了，blablabla，解决方案是在.ssh/config对应的Host项下面加上




```
StrictHostKeyChecking no
UserKnownHostsFile /dev/null
```


 


