---
title: "DNS解析的并发性"
date: 2020-04-06T15:03:00+08:00
draft: false
---

我以前一直以为 resolv.conf 中的nameserver是按照顺序解析的，今天才知道在某个glibc的版本以上，是并行解析的，仅当 options single-request 设置之后，才会一个一个按顺序来。但是我在使用go get 解析私有域名时，还是会解析错误，我猜想go的实现可能不一样，也没有去证实。最后当连接了公司的vpn时，使用脚本将dns设置为仅使用公司的dns就可以了


 


