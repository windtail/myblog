---
title: "MinGW(msys 1.0) 编译 splint"
date: 2011-05-27T23:55:00+08:00
draft: false
---

 


splint是一个开源的Lint工具，在mingw上编译还需要一点修改。


 


首先mingw(msys 1.0)中必须包含gcc g++ flex bison，其他我也不知道了，反正我当时把开发用的基本都装上了。


 


切换到解压的源代码目录


 


# ./configure


# 将生成的config.h重命名为winconfig.h，并在最后添加 #undef WIN32


# 打开osd.c文件，在#include "osd.h" 一行下面，加#define WIN32，在文件最后加#undef WIN32 （这个文件#define WIN32编译不过去）


# 随便找个c文件，添加 


    int yywrap() {


      return 1;


    }


 


    这是因为mingw的flex的库中未提供默认的yywrap函数，链接时会出问题。为了尽量少地修改文件，我把这个放osd.c文件的最后了。


# make


# make install


    默认安装在/usr/local/share的路径下，安装时不能成功设置环境变量LARCH\_PATH，可以手动在我的电脑->属性中添加环境变量LARCH\_PATH，值为 /usr/local/share/splint/lib对应的实际windows路径


 


完毕！


