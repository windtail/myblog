---
title: "luacom cygwin"
date: 2012-07-01T17:51:00+08:00
draft: false
---

前一段时间想用luacom来操作word文档，最终发现总有[那么点问题](http://blog.csdn.net/windtailljj/article/details/7203294)。这两天用cygwin写bash脚本来完成一些Word文档操作，不得已总要调用cscript，通过javascript来访问wordr com对象，而这样调用cscript有两个问题让我很不爽：
1. cscript又只接受Windows格式的路径，每次都要用 $(cygpath -w xxx) 来转换路径
2. cscript输出或错误都是gb2312的，每次都要转换成utf-8：cscipt //nologo xxxxx.js 2>&1 | piconv -f gb2312 -t utf-8


### Cygwin如何直接访问com对象呢？


cygwin有lua，而lua有luacom，这个能用吗？我刚才试了一下，基本功能貌似没有什么问题。因为cygwin本身就已经将文件名转换成了utf-8的，所有之前[一篇文章](http://blog.csdn.net/windtailljj/article/details/7203129)纠结于gb2312和utf-8的转换问题也就不是问题了。


### 编译安装luacom


cygwin需要安装 cmake make gcc g++ git lua，其他基础的当然也要  




* git clone https://github.com/davidm/luacom.git
* cd luacom
* mkdir build
* cmake ..
* make
* make install


### 测试FileSystemObject的枚举中文文件夹功能




```
#!/usr/bin/lua

require "luacom"

local fso = luacom.CreateObject("Scripting.FileSystemObject")
local myFolder = fso:GetFolder("F:")
for _,file in luacom.pairs(myFolder.SubFolders) do
  print(file.Name)
end
```


注：我的F:盘有很我中文文件夹，测试结果正确


### 测试Word文档建立和保存




```
#!/usr/bin/lua

require "luacom"

wordApp = luacom.CreateObject("Word.Application")
wordApp.Visible = true

wordDoc = wordApp.Documents:Add()
wordApp.Selection:TypeText("中文真的可以吗，我也不知道啊！")
wordDoc:SaveAs2("F:\\中文的文件名哦还挺长的.docx")

wordDoc:Close(0)
wordApp:Quit(0)

```
  


  




