---
title: "sphinx PDF 中文"
date: 2018-01-15T19:02:00+08:00
draft: false
---

使用reST撰写文档时，需要分多个文档时，就必须使用sphinx了，sphinx说起来很简单的，但是默认是不是支持中文的。幸好我出生的晚，sphinx现在已经支持xelatex了^\_^


安装
--


除了[pandoc那篇文章](http://www.cnblogs.com/windtail/p/8260193.html)里写到的要安装的东西，还需要安装如下软件包




```
sudo apt install latexmk texlive-lang-chinese
```


 


操作
--




```
sphinx-quickstart docs
```


以上命令会在当前目录的 docs 子目录中创建一大堆文件（如果不指定docs，则在当前文件夹内创建）。如果是英文的话，我们只要执行




```
make latexpdf
```


PDF文档就生成了，但是中文需要修改下配置，研究了半天，最简单的方式是




```
latex_engine = 'xelatex'
latex\_elements = {
 # The paper size ('letterpaper' or 'a4paper').
    #
    # 'papersize': 'letterpaper',

    # The font size ('10pt', '11pt' or '12pt').
    #
    # 'pointsize': '10pt',

    'fncychap' : '',

 # Additional stuff for the LaTeX preamble.
    #
    'preamble': r'''\usepackage{ctex}
 ''',

 # Latex figure (float) alignment
    #
    # 'figure\_align': 'htbp',
}
```


本来也想用xeCJK，不过搞起来有点困难（虽然也能生成PDF），这里只是引用入ctex宏包，然后把fncychap宏包给去掉了，因为它不支持中文，如果加上章节标题就会是“Chapter 1”这种，而不是“第一章”，当然，去掉之后，标题就不那么fancy了，不过那都是[latex的事](http://www.latexstudio.net/archives/413)了，我无能为力了！


 


