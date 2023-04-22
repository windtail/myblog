---
title: "emacs 简记"
date: 2018-01-04T16:38:00+08:00
draft: false
---

简介
--


Emacs作为神的编辑器，不用介绍了吧，说点感受。


用了一段时间了，总体感觉其实Emacs是很简单的，甚至比vim还简单，因为在X环境下，打开后可以就像记事本一样使用。但是，使用Emacs的人一般都是程序员，而Emacs要用来编程，总没有那些IDE好用，还总想把它折腾成好用的IDE，但又苦于没有一个适合自己的配置，总之，我现在把Emacs定位成IDE的补充，写几个小程序时，完全可以用，并且很方便，大的项目时，还是IDE更好一点，比如Pycharm，Eclipse等。但是，不管什么编程语言，基本上都可以在Emacs中找到插件，而有一些甚至IDE都支持得不好，比如reSTructuredText，另外在Linux环境中，目前还没有找到比Emacs中magit插件更好用的Git GUI工具了。


我的配置
----


作为一个对工具有要求的程序员，我当然不会放过每一个配置机会，^\_^，我今天把配置上传到了Github，希望对初学者有所帮助。


项目的地址是 <https://github.com/windtail/emacs-config>，大家看下，README就知道怎么用了，这里就不再说了。


reSTructuredText
----------------


reST在Linux下貌似没有好的编辑器，vscode也许还行（没怎么用过），sublime-text在Linux下中文输入都要搞半天，Emacs25默认就支持rst-mode，可以写reST，不过支持得也不好，主要是Emacs的人都用org了，reST不是Emacs世界的主流。好在我目前只用到很少的功能，也就两个快捷键：


* C-c C-= ：rst-adjust，在标题下输入三个符号，如---，再按这个，就会自动地补全到标题的长度
* C-c C-c C-c ：rst-compile，编译成html，需要用到 docutils


### 如何转换为人间的格式


reST虽好，但是非程序员他们不喜欢，大家一般都要pdf或者doc/docx，好在比较简单，建议使用virtualenv来管理转换需要的程序。


* $ sudo apt install virtualenvwrapper
* （关掉终端，重新打开一个）
* $ mkvirtualenv rst
* $ pip install docutils rst2pdf sphinx


以后要使用这个virtualenv，只要在终端中输入 workon rst即可


* rst2html.py 可以将reST转换为html
* rst2odt.py 可以将reST转换为odt格式，用open office打开后，可以另存为doc或docx格式，也可以另存为pdf格式
* rst2pdf 理论上可以将reST转换为pdf格式，但是在python3下貌似不能运行，有语法错误（暂时未用）


理论上我们还可以使用pandoc把reST转换为各种格式，比如pdf，但是转pdf需要tex支持，而tex中文还得搞半天，等有时间再学习。


