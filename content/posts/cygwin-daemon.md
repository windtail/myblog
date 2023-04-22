---
title: "cygwin daemon"
date: 2012-07-06T22:25:00+08:00
draft: false
---

前一段时间遇到一个问题，最近才解决，主要也是对cygwin不够了解。


  




### 问题描述


服务器端安装了cygwin的sshd服务器，想在git push后时执行一个时间比较长的脚本，当然不想在前台执行，所以采用后台执行，结果，当然是不行，执行一半就被强制关闭了。


### 问题分析


之前一直不明白为什么，后来想了想Linux上的东西，总算明白了，cygwin调用的也是bash，shell退出后，shell运行的程序就退出了。除非运行的是daemon  




### 守护进程


Linux下弄daemon，网上有很多，但是windows下机制不一样啊，没有什么init进程来接管无父进程的子进程。其实想想windows下的服务就是守护进程，再一搜，cygwin果然还是提供这种机制的。


cygrunsrv 可以将cygwin的程序安装成windows的服务，服务的stdout和stderr都被重定向到/var/log/.log文件，cygwin的具体用法参见帮助即可。


### 问题解决


编一个程序，每次接收到SIGUSR1时，启动运行一个shell脚本，做我想做的事情。然后在git repo的post-update钩子中，用kill向daemon发送消息


* signal处理类似标准linux，包含signal.h，然后用signal函数安装一个回调函数，注意事项也和Linux中一样，比如不要在signal函数中做IO操作之类的。可以这样做，主循环平时pause()，收到signal时，置个标志位，主循环检测到标志位后启动shell脚本
* 启动shell脚本关系到新建一个进程，linux中是fork，但是cygwin中使用fork太慢（官方文档说的），推荐用spawn()，这个函数的头文件在process.h（找了半天），用法参见MSDN，因为这实际是Windows的一个函数
* cygrunsrv默认采用SYSTEM用户来安装daemon，所以运行时会和直接在cygwin中有差别，主要是环境变量，因此可以在安装daemon时指定 --user  --password ，然后调用shell时再采用一个wrapper脚本，执行 bash --login
* 启动脚本后如果什么也不管，那脚本运行完后，会产生一个的僵尸进程，如果实在不想知道启动的脚本的运行结果，可以直接忽略SIGCLD，将其回调函数设置为SIG\_IGN，如果想了解脚本的返回值，就用waitpid或wait来获得返回值，这和linux下是一样的，不过我这次是直接忽略的SIGCLD，没有试waitpid
* post-update钩子中，kill如何找到daemon呢，cygwin又提供了一个ps程序可以找到daemon的pid，命令如下： kill -s SIGUSR1 $(ps -a | grep  | cut -f1 -d " ")

