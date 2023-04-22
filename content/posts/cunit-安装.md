---
title: "CUnit 安装"
date: 2011-02-28T22:59:00+08:00
draft: false
---

 


测试系统：Fedora 11


 


最新版的CUnit-2.1.2不能编译过去，貌似需要Ubuntu才行


 


下载CUnit-2.1.0，2006年更新的那个版本，解压，然后：


$ (autoreconf --install) 应该有这步，但是我看有configure文件，就没做这一步


$ ./configure


$ make


# make install #需要root权限


# vi /etc/ld.so.conf   


# 最后面加 /usr/local/lib


# ldconfig


 


测试安装是否正确：


将 http://cunit.sourceforge.net/example.html 页面的例子复制下来，保存为 t.c 文件


编译： gcc -o t t.c -lcunit


运行： ./t


结果：


  

     CUnit - A Unit testing framework for C - Version 2.1-0  

     http://cunit.sourceforge.net/  

  

  

Suite: Suite\_1  

  Test: test of fprintf() ... passed  

  Test: test of fread() ... passed  

  

--Run Summary: Type      Total     Ran  Passed  Failed  

               suites        1       1     n/a       0  

               tests         2       2       2       0  

               asserts       5       5       5       0


 


本文参考： http://www.55gps.com/html/6/3/20080109/2821.html


