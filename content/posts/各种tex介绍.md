---
title: "各种Tex介绍"
date: 2011-07-21T16:15:00+08:00
draft: false
---

一点并不正式的说明
---------

 *本帖最后由 milksea 于 2011-2-11 22:35 编辑*   
  
**补充：TUG 的说明 <http://www.tug.org/levels.html>**  
  
我作个简单的说明。  
  
**语言层面：**  
  
TeX 是一种**宏语言**。就像别的语言有库一样，TeX 语言有**宏**的集合。这些宏，就是用这个语言写出来的东西，供大家共用的。TeX 有个最基本的宏集合，与基础语言一起构成一种“**格式**”，就是 Plain TeX。基本的 TeX 语言和 Plain TeX 的宏，都是 Knuth 完成的。  
后来有了 LaTeX，就是 Lamport TeX。LaTeX 也是宏的集合，也构成一种与 Plain TeX 不一样的**格式**。这种格式提供了不少方便的功能，并强调结构化的文档，因而很快风行起来。  
世界在发展，LaTeX 这个格式提供的基本功能并不能满足所有人的需要，就有人接着写宏。这些宏可能就是在 LaTeX 这个格式的框架下面编写的，也就是说是依赖于 LaTeX 宏的宏。一些有能力有精力的人把他觉得有用的宏打成包，公开发布，就成为了 LaTeX 的**宏包**。换言之，宏包就是基本的格式的扩展。当然它是人人可写的，特别有名的，比如美国数学会官方定制的 AMSTeX/AMSLaTeX，就是一套 TeX/LaTeX 格式下面的宏包。  
  
**再来说软件层面：**  
  
一个语言是抽象的，不能运行就没有结果。因此 Knuth 在设计这个语言的同时也给出了一个程序用来把 TeX 语言的代码转换为排版的结果——这个程序当然也叫 TeX。嗯，可以把这个程序看做编译器。直接运行 tex 命令（我用小写字母表示你实际使用的命令），默认就是用 Plain TeX 这种格式进行排版。为示区别，我们可以把 Knuth 的这个 TeX 程序叫 Knuth TeX。  
用 tex 加上一个选择格式的命令行编译选项，就可以改用 LaTeX 这种格式进行排版了。但这很麻烦，于是就把 tex 命令与对应编译选项合成为一个命令，叫 latex。简言之，latex 命令就是 tex 命令加一个选项的简写方式。  
Knuth TeX 这个程序有一些功能不好实现，后面就有人进行扩展，得到 ε-TeX 这个程序，一般写成 eTeX。eTeX 程序和 Knuth TeX 都是 TeX 语言的一个实现，eTeX 增加了少量的几个命令，但一般来说是没有太多区别的。  
Knuth TeX 输出的格式是 DVI（DeVice Independent）文件，但后来电子出版业和电子文档交换中常用的格式是 Adobe 公司开发的 PostScript 格式（PS）和 Portable Document Format 格式（PDF）。因此就需要有一些工具完成这样的转换，一些转换程序应运而生：Dvips（把 DVI 转换为 PS）；DVIPDF、DVIPDFM、DVIPDFMx（把 DVI 转换为 PDF，可以认为后面的是前面的改进版）。  
转换的过程令人不爽，于是就又有了 TeX 语言的又一个实现，pdfTeX。它会把 TeX 语言写的代码直接编译成 PDF 文件。当然，不难理解 pdftex 命令就是用 pdfTeX 这个程序和 Plain TeX 这个格式进行排版，而 pdflatex 这个命令就是用 pdfTeX 这个程序和 LaTeX 格式进行排版。不过 pdfTeX 程序也保留了输出 DVI 格式的能力。  
时代在发展，多字节的编码渐渐代替 ASCII 成为主流。为了支持 Unicode 编码和直接访问操作系统字体，又出现了 TeX 语言的新的实现，即 XeTeX。作为一个现代的程序，XeTeX 也直接输出 PDF 文件（我们暂不去管它内部有格式转换的实现方式）。于是，不难理解 xetex 命令就是使用 XeTeX 程序以 Plain TeX 格式排版，而 xelatex 命令就是用 XeTeX 程序以 LaTeX 格式排版。  
哦，人们的要求总是在发展，现在又想在 TeX 中嵌入其他语言进行更强有力的扩展了。于是 Lua 脚本语言和 TeX 语言的结合体，LuaTeX 应运而生。LuaTeX 程序也是 TeX 语言的一个完整的有扩展的实现。LuaTeX 支持 Unicode、系统字体和内嵌语言扩展，能直接输出 PDF 格式文件，也可以仍然输出 DVI 格式。于是 LuaTeX 程序又对应了许多命令：luatex 使用 Plain TeX 格式，输出 DVI；lualatex 使用 LaTeX 格式，输出 DVI；pdfluatex 使用 Plain TeX 格式，输出 PDF；pdflualatex 使用 LaTeX 格式，输出 PDF。  
  
瞧，语言的脉络是简单的，但软件程序总是层出不穷。  
  
  
故事还没有完，前面我遗漏了一个重要的**格式**，叫做 ConTeXt。这个格式从一开始就很强调与脚本语言，也就是具体实现程序的结合。过去旧版本的 ConTeXt 是使用 pdfTeX 程序作为它的排版引擎的，用来扩展的脚本语言是 ruby，编译文件使用的命令一般是 texexec；新版本的 ConTeXt 则使用 LuaTeX 作为它的排版引擎，脚本也都直接使用 Lua，编译时使用的命令是 context。那么不难猜，所谓“XeConTeXt”是什么东西，它事实上是 ConTeXt 改用 XeTeX 程序作为它的排版引擎的一种编译方式，实际的命令则是 texexec 或 context 加上适当的命令行选项。  
  
  
最后说一下 BibTeX 和 MakeIndex。这两个都是与 TeX 相关联的工具程序，一般用在 LaTeX 格式上。BibTeX 处理 .tex 文件，根据其中的引用，从文献数据库中提取生成参考文献列表；而 MakeIndex 处理 LaTeX 格式编译时输出的 .idx 文件（里面是索引条目），生成 .ind 文件（里面是排序整理好的索引条目）。  
  
（是的，上面的叙述我还是略去了一些细节，东西很杂，不多说了。）

转自：<http://bbs.ctex.org/viewthread.php?tid=49647>  


  


  

