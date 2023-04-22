---
title: "luacom 中文 终极解决方案"
date: 2012-09-06T22:11:00+08:00
draft: false
---

经过之前的工作，总算可以在Cygwin中使用luacom了（[参见这篇文章](http://www.cnblogs.com/windtail/archive/2012/07/01/2623173.html)），但是不能在Windows下直接使用，有些事情还是很难办的。比如今天我想用luatex直接调用lua模块实现编译tex文档时，将其中包含的visio图在线转成pdf。


于是，我终于忍不了了，把luacom的源代码编译并调试了下，修正了其中的一个BUG，参见[https://github.com/windtail/luacom](https://github.com/windtail/luacom "https://github.com/windtail/luacom")的最新一次提交。


测试用例还是之前在cygwin中的那两个 [http://www.cnblogs.com/windtail/archive/2012/07/01/2623173.html](http://www.cnblogs.com/windtail/archive/2012/07/01/2623173.html "http://www.cnblogs.com/windtail/archive/2012/07/01/2623173.html")


### 使用注意事项


* 任何传递给com函数的中文字符串都必须是GBK编码，如果你喜欢用UTF-8的方式来写lua源文件，那么必须在调用前先用luaiconv转换成GBK
* 任何从com函数传出的中文字符也都是GBK编码的
* 在Cygwin中使用时，编码必须都是UTF-8的


有关我和luacom纠结的历史，参见我博客的其他文章，编译好的[luacom.dll可在此下载](http://files.cnblogs.com/windtail/luacom.rar)，直接覆盖luaforwindows中clibs文件夹下的文件即可（建议备份^\_^）


