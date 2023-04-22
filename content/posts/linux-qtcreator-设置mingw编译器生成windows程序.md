---
title: "Linux QtCreator 设置mingw编译器生成windows程序"
date: 2016-03-17T21:49:00+08:00
draft: false
---

Qt跨平台，那必须在Linux平台编译一个可以在windows下运行的Qt程序才行，当然还得和QtCreator环境弄在一起才行


工作环境：Centos 7


yum install qt5-qt\* ming32-qt5-qt\* ming64-qt5-qt\*       # 安装所有Qt的包


yum install qt-creator            # 安装qtcreator


**以mingw64为例**


打开QtCreator，options=> Build & Run => Qt Versions => Add


qmake location:  /usr/bin/x86\_64-w64-mingw32-qmake-qt5 (注意，点后面的Browser是不好使的，因为Browser时对话框要求文件名必须是qmake)


添加完Qt后，会提示没有对应的编译器，我们需要去添加Compiler选项卡中去添加一个编译器，不过在这之前，请点击qmake location下面一行最右边的Details，其中有一项 ABI:x86-windows-msys-pe-64bit


切换到Compilers选项卡，Add=>MinGW，在compiler path中填写 which x86\_64-w64-mingw32-g++的输出结果，而ABI就选上面记录的那个


最后在Kits选项卡中，Add，compiler和qt version就选刚加的，sysroot填 /usr/x86\_64-w64-mingw32/sys-root


大功告成！


mingw32类似，qmake localtion: /usr/bin/i686-w64-mingw32-qmake-qt5, compiler path: which i686-w64-mingw32-g++, sysroot: /usr/i686-w64-mingw32/sys-root


**发布**


发布这事，就是用[depends](http://www.dependencywalker.com/)查一下信赖，把该拷的拷过去就行了，实在不行就把 /mingw/bin和/mingw/lib/qt5/plugins文件夹中的所有文件都拷到可执行文件所在目录，然后慢慢删


 


<!--
p, li { white-space: pre-wrap; }
-->/usr/bin/i686-w64-mingw32-qmake-qt5
-->
<!--
p, li { white-space: pre-wrap; }
-->
