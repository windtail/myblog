---
title: "LaTeX 排版C语言代码"
date: 2011-08-24T00:16:00+08:00
draft: false
---

转载自http://blog.sina.com.cn/s/blog\_51e68f8d0100avil.html  




  




转载自http://blog.linuxgem.org/lyanry/show/319.html


listings 是专用于代码排版的 LaTeX宏包，可对关键词、注释和字符串等使用不同的字体和颜色或颜色，也可以为代码添加边框、背景等风格。


### 1 基本用法


下面给出一份用于排版 C 语言 HelloWorld 程序代码的完整的 LaTeX 文档：



\documentclass{article}  

\usepackage{listings}  

  

\begin{document}  

  

\begin{lstlisting}[language=C]  

int main(int argc, char \*\* argv)  

{  

  

printf("Hello world!\n");  

  

return 0;  

}  

\end{lstlisting}  

  

\end{document}

注意，要使用 listings 宏包提供的语法高亮，需要 xcolor 宏包支持。


语法高亮的排版效果如下图所示：


![](http://blog.linuxgem.org/user_files/lyanry/Image/tex/listing_2.png "备忘《三》latex <wbr>listings宏包使用")
### 4 添加边框


listings宏包为代码边框提供了很多风格，大体可分为带有阴影的边框与圆角边框。这里仅仅给出一个阴影边框的示例，至于其它边框风格，可查阅listings 宏包文档，里面给出了一些示例。


下面 LaTeX 源文档将为代码添加阴影边框，并将阴影设置为浅灰色：



\begin{lstlisting}[language={[ANSI]C},keywordstyle=\color{blue!70},commentstyle=\color{red!50!green!50!blue!50},frame=shadowbox,
 rulesepcolor=\color{red!20!green!20!blue!20}]  

int main(int argc, char \*\* argv)  

{  

  

printf("Hello world!\n");  

  

return 0;  

}  

\end{lstlisting}

排版效果如下图：


![](http://blog.linuxgem.org/user_files/lyanry/Image/tex/listing_3.png "Open image in new window")


### 5 添加行号


很多时候需要对文档中的代码进行解释，只有带有行号的代码才可以让解释更清晰，因为你只需要说第 x行代码有什么作用即可。如果没有行号，那对读者而言就太残忍了，他们不得不从你的文字叙述中得知行号信息，然后去一行一行的查到相应代码行。


listings 宏包通过参数 numbers 来设定行号，该参数的值有两个，分别是 left 与right，表示行号显示在代码的左侧还是右侧。下面为带有边框的代码添加行号，并设置行号字体为 \tiny：



\begin{lstlisting}[language={[ANSI]C},numbers=left,
 numberstyle=\tiny,keywordstyle=\color{blue!70},commentstyle=\color{red!50!green!50!blue!50},frame=shadowbox,
 rulesepcolor=\color{red!20!green!20!blue!20}]  

int main(int argc, char \*\* argv)  

{  

  

printf("Hello world!\n");  

  

return 0;  

}  

\end{lstlisting}

排版效果如下图所示：


![](http://blog.linuxgem.org/user_files/lyanry/Image/tex/listing_4.png "Open image in new window")
### 6 全局设置


上面所给的各个示例中，lstlisting 环境后面尾随了很多参数，要是每使用一次 lstlisting环境就要设置这么多参数，那就没什么意思了。


可以使用 \lstset 命令在 LaTeX 源文档的导言区设定好 lstlisting 环境所用的公共参数，如下：



\documentclass{article}  

\usepackage{listings}  

\usepackage{xcolor}  

  

\begin{document}  

  

\lstset{numbers=left,  

numberstyle=\tiny,  

keywordstyle=\color{blue!70},commentstyle=\color{red!50!green!50!blue!50},  

frame=shadowbox,  

rulesepcolor=\color{red!20!green!20!blue!20}  

}  

  

\begin{lstlisting}[language={[ANSI]C}]  

int main(int argc, char \*\* argv)  

{  

  

printf("Hello world!\n");  

  

return 0;  

}  

\end{lstlisting}  

  

\end{document}

### 7 显示中文


listings 宏包默认是不支持包含中文字串的代码显示的，但是可以使用 “逃逸” 字串来显示中文。


在 \lstset 命令中设置逃逸字串的开始符号与终止符号，推荐使用的符号是左引号，即 “ `”



\lstset{numbers=left,  

numberstyle=\tiny,keywordstyle=\color{blue!70},commentstyle=\color{red!50!green!50!blue!50},  

frame=shadowbox, rulesepcolor=\color{red!20!green!20!blue!20},  

escapeinside=``}  

  

……  

  

\begin{lstlisting}[language={[ANSI]C}]  

int main(int argc, char \*\* argv)  

{  

  

printf("`我爱中文`!\n");  

  

return 0;  

}  

\end{lstlisting}

### 8 调整一下边距


listings的代码框的宽度默认是与页芯等宽的，其上边距也过于小，可根据自己的审美观念适度调整一下。我通常是将代码框的左右边距设置为2em，上边距为 1em，下边距采用默认值即可，所作设定如下：



\lstset{numbers=left,numberstyle=\tiny,keywordstyle=\color{blue!70},commentstyle=\color{red!50!green!50!blue!50},frame=shadowbox,
 rulesepcolor=\color{red!20!green!20!blue!20},escapeinside=``,xleftmargin=2em,xrightmargin=2em, aboveskip=1em}

  

