---
title: "luacom GetEnumerator 不好使？"
date: 2012-01-15T23:37:00+08:00
draft: false
---

上一篇说了luacom不支持gbk，不过可以用iconv来解决，但是我还发现了一个问题，貌似 enumerator 不太好使



```
require "luacom"
require "luaiconv"

function createIconv(to, from)
	local cd = iconv.new(to, from)
	return function(txt)
		return cd:iconv(txt)
	end
end

L = createIconv("utf-8", "gbk")
Z = createIconv("gbk", "utf-8")

fs = luacom.CreateObject("Scripting.FileSystemObject")
fd = fs:GetFolder("F:\\temp")

fenum = luacom.GetEnumerator(fd.Files)

f = fenum:Next()
while f do
	print(Z(f.Path))
	print(fs:FileExists(f.Path))
	f = fenum:Next()
end
```



（上面的代码，改用luacom.pairs枚举也有同样问题）


  

这个代码假设 F:\temp下面有中文文件名的文件，发现 Z 函数对中文文件名文件，总是返回错误，而 Enumerator获得的路径名，直接判断是不是存在，居然是false！


这是什么问题呢，同样的方式，用javascript实现，完全没有问题啊。


  




**求高手解答**！ bug？？（本人实在不了解COM，没有勇气看luacom源代码）


  




  

