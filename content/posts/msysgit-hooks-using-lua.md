---
title: "msysgit hooks using lua"
date: 2012-01-02T15:06:00+08:00
draft: false
---

背景
==


最近想在公司整一个git服务器，需要做一些配置，看着.git/hooks/文件夹中的\*.sample文件夹，很是不解，把".sample"去掉就可以运行？


事实证明，真的可以，将下面的代码放到pre-commit文件中，就可以在commit时，输出"Hello Git!"消息  





```
#!/bin/sh
echo "Hello Git!"
```

Lua
===


虽然我会一点点bash的脚本，但其实跟不会没多大区别~~


鉴于Lua的简单易用性，及在Windows平台采用wxLua做界面是如此地优雅，已经准备在以后的日常工作中广泛采用Lua进行一些简单地自动化操作。


所以，我希望用lua脚本来写git hooks，首先我们得安装lua，


* 资源主页：<http://code.google.com/p/luaforwindows/>  （可能被墙了）
* 下载地址：<http://luaforwindows.googlecode.com/files/LuaForWindows_v5.1.4-45.exe>


然后，这样做写 pre-commit 文件即可



```
#!/bin/env lua.exe
print "Hello Git From Lua!"
```

**注意是lua.exe，而不是lua**


ps：我也只是开了个头，貌似在调用git log来获得各种信息时会有点费劲，不过等我试了再说~


其他链接
====


msysgit：<http://msysgit.googlecode.com/files/Git-1.7.8-preview20111206.exe>  




tortoisegit：<http://tortoisegit.googlecode.com/files/TortoiseGit-1.7.6.0-32bit.msi>


Tortoise Redmine Plugin：<http://redmine-projects.googlecode.com/files/TortoiseRedminePlugin_32bit_1.1.0.8.msi>  




