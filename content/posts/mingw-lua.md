---
title: "mingw lua"
date: 2012-07-31T22:27:00+08:00
draft: false
---

前天尝试编cygwin上的lua模块（参见[上一篇文章 cygwin install lua modules](http://blog.csdn.net/windtailljj/article/details/7800084)），累死了也没把gui搞定，iup有编译好的，但是不知道怎么用，wxLua编译不过。


其实我主要还是用cygwin来做开发，用Lua写一些脚本方便开发，所以今天转战mingw/**msys**，mingw真是好啊，与win32真是无缝连接啊，lua和库都不用自己来编了，把LuaForWindows安装完的文件 lua.exe wlua.exe bin2c.exe lua和clibs文件夹，拿来放/usr/local/bin中基本就行了，可能还有两个需要做：


1. 执行 lua.exe -e "print(package.path)" 和 lua.exe -e "print(package.cpath)" 看看是啥，如果不对适当调整下 lua和clibs文件夹的位置
2. 将clibs文件夹放到path环境变量中，Windows会在环境变量中找dll


比cygwin简单多了！另外，一些开发常用的 cmake doxygen等等，直接下载Windows的zip文件即可，解压后放 bin 文件中就行了！


对lua来说文件的路径名也好处理多了，比如以下文件（test.lua）可以msys中这样执行 **lua.exe test.lua /g/a.txt**  





```
filename = ...
print(filename)

f = io.open(filename, "w")
f:write("Hello MinGW")
f:close()

```
  

运行结果是： g:/a.txt


并且 g:\a.txt 正确建立


要是cygwin可就悲剧了，因为Windows中的程序不能处理 /cygdrive/g/a.txt 这样的路径名  




