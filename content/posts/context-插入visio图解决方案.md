---
title: "ConTeXt 插入Visio图解决方案"
date: 2012-09-08T13:19:00+08:00
draft: false
---

### 背景


根据CTeX论坛里的帖子[1](http://bbs.ctex.org/forum.php?mod=viewthread&tid=43683) [2](http://bbs.ctex.org/forum.php?mod=viewthread&tid=64499) TeX文件中如果要插入visio图，必须先转换成PDF格式，可以手动转换，也可以用VBS批量转换。我最近在研究ConTeXt，不了解LaTeX，我就说说ConTeXt的问题吧。


1. 首先，VBS转换成PDF是外挂的，转换过程独立于TeX的编译过程，必须手动写一个Makefile建立依赖关系，如果文档中包含的Visio图变了，那就必须修改Makefile，当然也可以编一个程序扫描所有的TeX文档，提取visio图信息，自动建立依赖关系。如果能够让TeX编译时在线地将visio图转成pdf，然后包含进来岂不是更好。
2. 其次，大家给出的解决方案对于PDF切白边一般都要调用外部程序，imagemagick或pdfcrop等，或手动在visio中操作，并没有给出自动化的VBA程序。


### visio模块


LuaTeX确实是一个很好的想法，既然TeX可以调用Lua程序，而[LuaCom](http://www.cnblogs.com/windtail/archive/2012/09/06/2674260.html)又可以通过com接口调用visio的函数，自动转换就不是问题了！


插入外部图片时ConTeXt一般用下面的形式：




```
\useexternalfigure[figurename]
\placefigure[here][fig:demo]{caption}{\externalfigure[figurename][factor=...]}
```


visio模块的目标就是将externalfigure换成visio，即：




```
\usevisio[visioname]
\placefigure[here][fig:demo]{caption}{\visio[visioname][factor=...]}
```


 


想法很简单，就是在\usevisio时将visio转换成pdf，并记录pdf的文件名，然后内部调用下\useexternalfigure，\visio时当然是内部调用下\externalfigure啦。在编写visio模块时遇到并解决了如下几个问题：


* luacom不支持gb2312的中文路径，修改下luacom源代码即可，参见我的[上一篇博文](http://www.cnblogs.com/windtail/archive/2012/09/06/2674260.html) 和 [这个讨论](https://github.com/davidm/luacom/pull/6#issuecomment-8336246)
* \useexternalfigure和\externalfigure不支持中文的路径名，经我测试PDF文件名如果是中文的ConTeXt找不到文件，这个问题应该和luacom的问题相同。解决办法是将中文文件名转换成英文件文件名，一开始是想让用户在\usevisio时指定一个英文替代名，然后在\visio时使用那个英文名，想来想去觉得这样不好，于是我使用了MD5将中文名替换它的MD5值，以保证中文名对应的英文名的唯一性。
* 为了让ConTeXt搜索到转换完的PDF，必须将PDF文件夹加入到\setupexternalfigures设置的directory中，这是通过命令\setupvisiodirectory[pdf][PDF输出路径]来完成的，PDF输出路径可以是绝对也可以是相对，如果是相对路径那么会在设置时展开为相对于当前文件夹的绝对路径。由于\setupexternalfigures没有增量设置directory的功能，而我又不想在调用\setupvisiodirectory后用户之前设置的搜索路径就没了，于是我翻了下ConTeXt代码，发现搜索路径存储在figures.paths的table中，\setupvisiodirectory时将以前的路径保存下来了，如果用户在调用\setupvisiodirectory后想再修改搜索路径，那么就必须保证包含PDF输出路径
* 为了保证文档的结构性，visio图和其他图一样，也需要一个搜索路径，这个搜索路径暂时和其他是独立的。默认时会搜索 “.”、“..”、“.\image”、“.\figure”、“.\visio”，用户也可以加入全局性的文件夹列表，采用\setupvisiodirectory[visio][…,…,…]的方式
* 注意\setupvisiodirectory命令输入的文件夹都是分隔符都是“/”，因为“\”是TeX的特殊字符
* 我们写中文ConTeXt文档时，一般采用UTF-8格式，因此\usevisio和\visio收到的到文件名都是UTF-8的中文，但是Windows中文系统的文件名是GB2312的编码（这是为什么ConTeXt找到不中文文件名的原因），我们可以调用lua-iconv来搞定这个问题。visio模块默认是写给咱们自己用的，不过为了让外国人也能用上，我增加了一个设置命令\setupvisioencoding[source=…,system=…]，这个命令一看就明白，source是设置ConTeXt源代码的编码，而system是设置系统文件名的编码。
* \usevisio的完整格式是\usevisio[VISIO文件名（不要.vsd）][left=…,right=…,top=…,bottom=…]，第二个参数可选的，PDF切边的四周留白（margin），单位固定为mm（用命令时不要带单位，只用数字），如果忽略则默认都是0。我还不知道怎么在TeX中将dimension的单位转成mm，又不想自己用lua转换，所以单位不能用别的啦（貌似有LuaTeX提供了函数，有空再研究）。另外由于PDF文件名只和visio文件名有关，而与margin无关，因此一个visio图只能取一种margin，如果有需要可以修改下。
* 如果visio图被转换过一次后，visio图没有更新，就不会再转换了，这样可以加速后面的编译生成过程。不过这样又带来了问题，就是如果修改了margin但是没有修改visio图，PDF就不会更新，margin修改不会生效，办法就是touch下visio文件。
* 目前\usevisio时，没有将图形的尺寸整上来。在转换时获取图形的尺寸是很容易的，但是ConTeXt运行很多次，图形却只转换一次，所以如果每次要尺寸，那么就得整一个临时文件保存这些值，暂时还没做，原大小插入到文档中貌似没有问题。
* visio的VBA和word的VBA相差甚远，虽然我对word的VBA比较熟，但是visio的VBA真是一窍不通，文中操作visio的函数是手动操作时录制的宏，然后照着帮助修改了下，只在visio 2010中测试通过，不知道别的版本是否好使。


### 安装和使用


和zhfonts一样，这个模块放到了github上，大家可以用git（[git://github.com/windtail/ConTeXtVisio.git](git://github.com/windtail/ConTeXtVisio.git "git://github.com/windtail/ConTeXtVisio.git")）或直接从网页（[https://github.com/windtail/ConTeXtVisio/zipball/master](https://github.com/windtail/ConTeXtVisio/zipball/master "https://github.com/windtail/ConTeXtVisio/zipball/master")）下载最新版本（大家可以看看我写的[英文说明](https://github.com/windtail/ConTeXtVisio)），安装步骤如下：


* 下载[LuaForWindows](https://luaforwindows.googlecode.com/files/LuaForWindows_v5.1.4-46.exe)，并安装，将LuaTeX的模块搜索路径设置到luaforwindows的路径中，也可将Luaforwindows的模块拷贝到LuaTeX的搜索路径中，我选择了后者，因为这样简单。要想知道LuaTeX的搜索路径，用ConTeXt编译下clibs文件夹中的luapath.tex文件即可。（对于ConTeXt minimal来说是将 LuaForWindows安装路径中的 \5.1\lua\\* 和\5.1\clibs\\* 的全部内容拷贝到 \tex\texmf-mswin\bin\lib\lua\ 文件夹下）
* 将ConTeXtVisio.git中的clibs文件夹中的3个dll也复制到cpath（\tex\texmf-mswin\bin\lib\lua\）中（覆盖）
* 将ConTeXtVisio.git中的visio文件夹整个复制到 \tex\texmf-local\context\third\ 文件夹下（和zhfonts并列即可）
* 在ConTeXt的命令行中运行luatools –generate
* 打开visio 2010，文件=>选项=>信任中心=>信任中心设置=>宏设置=>启用所有宏=>确定
* 编译下visio/test文件夹中的testvisio.tex，应该是可以直接生成PDF的（生成PDF时，有时候LuaTeX会挂掉，暂时不知道是为什么，幸好这种情况貌似比较少）


### 心得小记


这段时间研究ConTeXt略有心得，简记在此，但愿后续的同志不用再走弯路。


学习ConTeXt首先第一件事就是阅读下[LiYanrui](http://garfileo.is-programmer.com/) 同学的博文，不然对ConTeXt都提不起兴趣，读完之后就可以开始尝试下排版中文了。


当弄了一段时间之后，就会发现不懂TeX是不行的，此时可以考虑阅读下*The TeXBook*，虽然Hans建议阅读 *A Beginner's Book of TEX* ，不过这本书基本已经绝迹江湖，只有亚马逊上有卖，价格不菲，搜了下图书馆，只发现国图和清华大学有。相比起来 *TeXBook* 更好从网上下到电子版的，甚至有中文译本。扫一扫简单的就行，以后用到时再慢慢看吧。（*TeX by topic* 网上也可以下到电子版的，不过没看过，不做评论）


学习Lua，看看 Programming in Lua （中文名：Lua 程序设计 第2版）这本书，非常好。相比TeX，Lua要好学多了，对不懂TeX的人来说，有时候真不太清楚TeX是怎么展开的，各种 \unexpand 、 \expandafter 到底是怎么工作的，不过Lua可以弥补你，把东西都放 \ctxlua{}中，一切由Lua程序来搞定就行！


看contextref文档，有问题多查查wiki和mailing list，尤其是wiki经常更新，会告诉你哪些功能和命令已经变了，不再好使了之类的。比如你看contexref上都写用tabulate，但是查下wiki你会发现这个早就不推荐，现在都用 \bTABLE \eTABLE 了。


这时，再读读zhfonts的源代码，你会发现其实还行，不是特别地难。大概就是把代码放一个Lua文件中，用\ctxloadluafile 加载这个文件，最后定义一些TeX命令用\ctxlua调用你的lua函数即可。关于写Lua模块，有几点经验：


* Lua本身支持模块化定义，简化模块中的函数定义，在文件头使用module声明模块，如：module("visio", package.seeall)，然后模块内部声明函数时不需要再加模块名了，自动都放到了指定模块名的table中，这一点在上面提到的那本书里有介绍
* 如果模块有语法错误，LuaTeX会忽略掉整个模块代码，最终的结果是导致你的命令未定义，所以最好在此之前自己测试下模块是否正确。安装了LuaForWindows之后，可以用SciTE编辑，调试代码，模块代码可以测试context函数是否为nil，如果为nil则表示在调试，这时可以自己实现一个简单的context函数，看看各种输出是否正确（参见visio.lua）
* 为了方便查看信息，经常会用\writestatus来输出信息，不过这也是很容易出bug的地方，因为输出时经常会忘记输出是给TeX的，一些特殊的TeX符号会导致问题，比如windows下的文件夹分隔符会导致未定义的TeX命令，最好编个函数，将特殊符号转换下，包括“% \ # { } [ ]”等等，看情况需要。
* 实在解决不了的问题，可以考虑搜索ConTeXt源代码 \tex\texmf-context\tex\context\base\\*.mkiv 及 lua文件，简单看一看可以获得意想不到的收获，比如我想知道figure的搜索路径存哪里了，看了代码之后发现存 figures.paths里，用 \ctxlua{print(table..concat(figures.paths, “,”))} 看下果然是！由于lua语言是动态的，如果想修改原来函数的功能，直接将原函数保存下来，替换掉它，然后在某些条件时才调用原来的函数即可，这一点LiYanrui在他的博文中也提到了。


### 有用的链接


* [system macros](http://wiki.contextgarden.net/System_Macros) ：Taco写的，读下这个，好处大大的有，一些别处不说的，这里都说了，比如 \unprotect \protect \dodoubleempty \processaction \getparameters \writestatus 等等


### 后记


我正在构思一个 MS Word 的模块，目的是为了方便从Word移植过来，不过现在又卡到字体上了，还得研究研究。


