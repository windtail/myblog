---
title: "CMake-5 调试"
date: 2012-03-06T19:55:00+08:00
draft: false
---

CMake-5 调试
----------


转自 <http://cttmayi.blog.cd/2011/01/23/cmake-5-%E8%B0%83%E8%AF%95/>  




  




    调试makefile,感觉比较好用的一个命令就是make --just-print (及make -n).他们打印makefile的整个流程,可以协助分析编译过程.


虽然CMake也产生makefile,来完成编译的工作,但产生的makefile中频繁的调用[cmake](http://cttmayi.blog.cd/tag/cmake/ "View all posts in cmake") -E([cmake](http://cttmayi.blog.cd/tag/cmake/ "View all posts in cmake")的命令模式),导致你如果不熟悉命令的细节的话,用make
 -n命令也无法了解其中编译的过程.


于是还是我还是先开始尝试[cmake](http://cttmayi.blog.cd/tag/cmake/ "View all posts in cmake")自己的一些调试命令,或许比较容易


1. cmake --debug-output :


既然cmake的debug模式. 打印想stack一样的cmake过程,同时会一写关于错误的信息.


测试的结果,看到一些cmake的流程.协助我找到一个问题(ADD\_SUBDIRECTORY(main maths),不能这样写,需要分开来写,否则只能找到一个,还不会报错,郁闷)


2. cmaek --debug-trycompile


不会删除try compile 目录.


还没发现有什么用.


3. trace:


进入cmake trace模式.打印出整个命令的流程


测试的结果. 暂时来看,细节太多.


4. make VERBOSE=1


在make执行的时候,显示出编译参数细节等,方便查询是否有错误


  

