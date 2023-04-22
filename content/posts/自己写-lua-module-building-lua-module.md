---
title: "自己写 lua module （Building lua module）"
date: 2012-01-15T21:28:00+08:00
draft: false
---

背景
==


学了一段时间lua，由于luacom不支持gbk，所以想做一个gbk转换成utf-8的模块。但是不会写模块怎么办？学！  

目标
==


创建一个calc模块，输出两个函数 sum和average，最后在lua程序想这样用：  

  


```
require "calc"
a = 5
b = 10
print(calc.sum(a,b))
print(calc.average(a,b))
```
  

  

学习过程简述
======


* 读Programming in Lua有关C API一节
* 读[Lua Reference Manual](http://www.lua.org/manual/5.1/) -> [The Application Program Interface](http://www.lua.org/manual/5.1/manual.html#3) 一节
* 下载lua5.1.4源代码，查看标准库 \*lib.c 文件是怎么写的
* 查看require函数源代码


学习总结
====


require("xxx") 函数在调用C模块时，会在cpath中查找 xxx.dll文件，若找到则调用 luaopen\_xxx函数， luaopen\_xxx 函数也必须 luc\_CFunction 类型的。
用户可以在luaopen\_xxx函数中用capi做任何事情，但一般是用luaL\_register注册一组函数到一个table中，并返回这个table  




  

配置lua编译环境
=========


  

1. 安装[luaforwindows](http://code.google.com/p/luaforwindows/)
2. 打开vs2008，工具->选项->项目和解决方案->VC++目录
	* 右上角选择 **包含文件**，添加luaforwindows安装目录中的include目录，比如 C:\Program Files\Lua\5.1\include
	* 右上角选择 **库文件**，添加luaforwindows安装目录中lib目录，比如 C:\Program Files\Lua\5.1\lib


生成calc模块
========


  

新建工程
----


打开vs2008，新建 -> 项目 -> Visual C++ -> Win32 -> Win32项目 -> 输入项目名 calc


应用程序类型：DLL  

附加选项：空项目  

  

添加库引用
-----


右上角选择 **所有配置**  

配置属性 -> 链接器 -> 附加依赖项 -> 输入 **lua51.lib**  

  




添加源文件
-----


添加一个新的文件 calc.**c** （注意是 .c 不是.cpp ），源代码如下：




```
//包含3个lua头文件
//如果本文件是cpp文件，则不要包含下面3个文件，而包含 lua.hpp 文件
#include 
#include 
#include 

static int sum(lua\_State \*L) {
 lua\_Number a = luaL\_checknumber(L, 1); //检查第1个参数是数字，如果是则返回这个数字
 lua\_Number b = luaL\_checknumber(L, 2); //检查第2个参数是数字，如果是则返回这个数字

 lua\_pushnumber(L, a+b); //返回值
 return 1; //函数返回值个数
}

static int average(lua\_State \*L) {
 lua\_Number a = luaL\_checknumber(L, 1); //检查第1个参数是数字，如果是则返回这个数字
 lua\_Number b = luaL\_checknumber(L, 2); //检查第2个参数是数字，如果是则返回这个数字

 lua\_pushnumber(L, (a+b)/2); //返回值
 return 1; //函数返回值个数
}

static luaL\_Reg module\_functions[] = {
 {"sum", sum}, //函数名，函数
 {"average", average},
 {NULL, NULL} //结尾
};

//注意用\_\_declspec(dllexport)输出函数
//如果本文件是 cpp 文件，则必须加extern "C"
\_\_declspec(dllexport) int luaopen\_calc(lua\_State \*L) {
 luaL\_register(L, "calc", module\_functions); //创建一个全局的calc table，所有函数都会按照指定名称注册成calc table的成员函数
 return 1; //luaopen\_calc也是一个luc\_CFunction，返回返回值的个数，返回值是由luaL\_register压入栈的
}
```
  


生成并测试
-----


生成release版的calc.dll，然后在calc.dll所在目录，新建一个 test.lua文件，把本文目标（上面）中的lua源程序拷进去，运行！结果正确！！


如果需要长期使用calc.dll，则可以将它拷贝到luaforwindows的clibs文件夹中，比如C:\Program Files\Lua\5.1\clibs


  




到此为止，我们已经掌握了用VS写lua C模块的基本方法了，下一篇文章，讲讲自己让luacom能够打开中文的word文件。


  




  

