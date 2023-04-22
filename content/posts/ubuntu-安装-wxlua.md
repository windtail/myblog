---
title: "Ubuntu 安装 wxlua"
date: 2012-05-13T22:15:00+08:00
draft: false
---

新立得中没有wxlua，所以只能下载最新的源代码来编译，幸好ubuntu中有wxWidgets不然这个也得自己来编译～～


编译很简单：


./configure


make


make install


结果很残酷：error while loading shared libraries: libwxlua\_gtk2u\_wxbindxrc-2.8.so.0: cannot open shared object file: No such file or directory


为啥呢，找找 libwxlua\_gtk2u\_wxbindxrc-2.8.so.0 在哪里，在 /usr/local/lib 中，而wx.so在/usr/local/lib/lua/5.1/中，wx.so找不到libwxlua\_gtk2u\_wxbindxrc-2.8.so.0


解决方法：很愚蠢，但好用，呵呵：


mv wx.so ../../wx.so


ln -s /usr/local/lib/wx.so wx.so


执行下 lua -e "require 'wx' "，没打印错误消息，表示正确！


  




顺便说下Ubuntu下安装的lua很干净，没有多余的模块，不像luaforwindows什么都装好了。好在可以用新立得安装luarocks，然后自己来安装各模块，比如


luarocks install luafilesystem 等等！  




