---
title: "Pandoc PDF 中文"
date: 2018-01-10T18:36:00+08:00
draft: false
---

最近终于**又决定**（^\_^）使用reStructuredText写文档了，输出PDF时的中文问题必须要解决下。


安装环境
----




```
sudo apt install texlive texlive-latex-extra texlive-latex-recommended texlive-xetex pandoc
```


非Debian或Windows请自行google之


LaTeX中文
-------


这个PDF的中文问题，其实就是LaTeX的中文问题，因为所有的PDF生成方式，都是先生成TeX和LaTeX文件的。


在XeTeX问世之前，在TeX里搞中文是一件比较费劲的事，好在XeTeX已经存在很多年了，XeTeX解决了Unicode和字体问题，但是排出来的中文断行，标点处理上还不太好，于是国人就写了[xeCJK](https://www.ctan.org/pkg/xecjk)这个宏包，概括一下，XeTeX+xeCJK之后，TeX中写中文就和写英文差不多友好了。


在linux-wiki.cn上有一篇写[LaTeX中文](http://linux-wiki.cn/wiki/LaTeX%E4%B8%AD%E6%96%87%E6%8E%92%E7%89%88%EF%BC%88%E4%BD%BF%E7%94%A8XeTeX%EF%BC%89)的文章，略微有一点老，不过大意是没有变化，我把那的代码粘贴到这里




```
\documentclass[11pt]{article}
\usepackage[BoldFont,SlantFont,CJKsetspaces,CJKchecksingle]{xeCJK}
\setCJKmainfont[BoldFont=SimHei]{SimSun}
\setCJKmonofont{SimSun}% 设置缺省中文字体
\parindent 2em   %段首缩进
 
\begin{document}
\section{举例}
\begin{verbatim}
标点。
\end{verbatim}
 
汉字Chinese数学$x=y$空格
\end{document}
```


根据[xeCJK的文档](http://mirrors.ctan.org/macros/xetex/latex/xecjk/xeCJK.pdf)，我认为以上的代码需要改成现在这个形式，如果文档是正确的话




```
\documentclass[11pt]{article}
\usepackage[AutoFakeBold=true,AutoFakeSlant=true,CJKspace=true,CheckSingle=true,PunctStyle=kaiming]{xeCJK}
\setCJKmainfont[BoldFont=SimHei]{SimSun}
\setCJKmonofont{SimSun}% 设置缺省中文字体
\parindent 2em   %段首缩进
 
\begin{document}
\section{举例}
\begin{verbatim}
标点。
\end{verbatim}
 
汉字Chinese数学$x=y$空格
\end{document}
```


注：xeCJK现在是CTeX-kit中的一员，在[Github上有仓库](https://github.com/CTeX-org/ctex-kit)


Pandoc的LaTeX
------------


pandoc可以转换很多格式，其中就包含reST，使用xelatex转换为PDF的命令行，基本格式为




```
pandoc -t latex --latex-engine=xelatex -s -o xxx.pdf xxx.rst
```


直接执行这条命令，当然是不好使的，因为没有使用xeCJK，也未指定中文字体。


pandoc在转换latex的时候，会使用一个默认的模板文件，这个模板文件可以使用如下命令查看




```
pandoc -D latex
```


我们当然可以使用自己的模板，具体参考[pandoc文档](http://pandoc.org/MANUAL.html)，不过我发现如果要求不高，默认的模板也是可以输出中文的，需要定义两个variable，我们这里直接使用命令行的方式传递变量值




```
pandoc -t latex --latex-engine=xelatex -s -VCJKoptions=BoldFont="SimHei" -VCJKmainfont="SimSun" -o xxx.pdf xxx.rst
```


今天就到这里，后面继续研究使用rst2pdf和sphinx来将reST文档转换为PDF


 


