---
title: "luacom打开中文路径的Word文件"
date: 2012-01-15T22:21:00+08:00
draft: false
---

背景
==


        luacom是一个非常强大的模块，它使我们可以应用各种com组件，比如Word，但是，有一个问题，中文文件名它不识别。为什么呢？因为com内部是unicode的，于是luacom要求所有输入都是utf-8的，而且luacom的输出也是utf-8的。这可肿么办啊？


iconv
=====


        GNU有个libiconv库，要是有这个我们就不怕了！ luaforge上搜索下，果然有lua-iconv，安装！


        luarocks install lua-iconv  不好意思，出错啦！出错的原因有两个：


* 我们没有安装libiconv库
* lua-iconv没有提供用cl编译的方法


自己编译lua-iconv
=============


1. 下载编译好的Windows版的 [libiconv](http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.9.1.bin.woe32.zip)
2. 下载 [lua-iconv](http://files.luaforge.net/releases/lua-iconv/lua-iconvSourcecode/lua-iconv-6/lua-iconv-6.tar.gz) 源代码
3. 解压libiconv-1.9.1.bin.woe32.zip 文件，并将include目录添加为vs2008的**包含文件**目录，将lib目录添加为vs2008的**库文件**目录中（[参见上一篇文章](http://blog.csdn.net/windtailljj/article/details/7203103)）
4. 解压lua-iconv的源代码
5. vs2008新建一个空的Win32 DLL工程（[参见上一篇文章](http://blog.csdn.net/windtailljj/article/details/7203103)），命名为luaiconv，将 luaiconv.c 文件添加到工程中
6. 修改源代码：int luaopen\_iconv(lua\_State \*L)  -> \_\_declspec(dllexport) int luaopen\_luaiconv(lua\_State \*L)
	* 注意：最后生成的dll文件名，必须和 luaopen\_luaiconv 中的 luaiconv 一样（[参见上一篇文章](http://blog.csdn.net/windtailljj/article/details/7203103)）
7. 项目属性 -> 链接器 -> 输入 -> 附加库文件 ： lua51.lib iconv.lib charset.lib
8. 编译生成release版的 luaiconv.dll 文件
9. 将luaiconv.dll文件，以及libiconv-1.9.1.bin.woe32.zip解压出来的 iconv.dll（知道我为什么要改luaopen\_iconv函数名了吧）和charset.dll文件一起拷贝到 luaforwindows的clibs目录中


测试
==




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

-- 注意：运行本文件会修改 C:\你好word.docx 文件，请注意备份

wordApp = assert(luacom.CreateObject("Word.Application"))
wordApp.Visible = true

wordDocPath = L"C:\\你好word.docx"
if not pcall(function() wordDoc = wordApp.Documents:Open(wordDocPath) end) then
	wordDoc = wordApp.Documents:Add()
end

wordApp.Selection:TypeText(L"你好word")
wordDoc:SaveAs2(wordDocPath, wdFormatDocument)
```
  


以上测试代码，第一次运行时会创建 C:\你好word.docx 文件，以后再运行时会打开这个文件，每次运行都会输入 “你好word” 文字。如果你是Word 2003，那么，请将docx改为doc即可。


参考文献
====


<http://hi.baidu.com/nivrrex/blog/item/17c231adad9e8a0f4b36d6ca.html>



> 
> 这位大哥自己用VC的函数写了转换函数，不过没有封装成库，而且我觉得写得不够简洁，用iconv库比较好，还不容易出错
> 
> 
> 


<http://www.cppblog.com/darkdestiny/archive/2009/04/25/81055.html>



> 
> 这位大哥，自己用iconv实现了转换，也没有封装成库。我的“L”函数也是从他这借来的，非常感谢！不过，我认为它这个相比我这个有两个弱点：
> 
> 
> 1、每次调用L函数，都要经过 iconv 打开、转换、关闭的过程，而我对一种形式的转换只需要打开一次（lua-iconv实现的^\_^）
> 
> 
> 2、如果要实现反向转换，即utf-8到gbk，那么还得修改模块，而我这里就不用了（当然也是lua-iconv实现的^\_^）  
> 
> 
> 
> 
> 


[libiconv](http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.9.1.bin.woe32.zip)的说明  





> 
>   
> 
> 
> 
> 
> 


  

