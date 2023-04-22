---
title: "doxygen lua"
date: 2012-05-10T23:43:00+08:00
draft: false
---

写C代码时，用惯了doxygen，现在写lua代码，也特别地希望用doxygen，上官网看了看，真发现有lua的filter


git clone https://github.com/alecchen/doxygen-lua.git 这个源即可获取，不过貌似人家没有管Windows什么事，perl脚本，所有东西都是Linux的


好在我们有cygwin，在cygwin中安装上perl和doxygen后，依然不能按照他说的办法安装，错误信息大约是找不到 Install.pm之类的，


经研究发现，其实是可以这样的：


* cp -r doxygen-lua/lib/Doxygen /lib/perl5/5.10/ 将整个Doxygen文件夹拷贝到perl的库文件夹中
* perldoc Doxygen::Lua 如果有输出帮助信息，说明拷贝正确
* cp doxygen-lua/bin/lua2dox /usr/bin/  这是最终要用到的脚本，放path中去
* ```
在Doxyfile中加上这么一句：FILTER_PATTERNS = *.lua=lua2dox 即可
```


验证：


* cd doxygen-lua/example
* 修改  FILTER\_PATTERNS = \*.lua=../bin/lua2dox => FILTER\_PATTERNS = \*.lua=lua2dox
* doxygen  执行doxygen
* 查看html


貌似脚本的功能有限。而且我觉得既然是Lua的Filter，就应该用Lua来写，这样也好让人加入，现在用perl，让我这个完全不懂perl的情何以堪啊~~  




