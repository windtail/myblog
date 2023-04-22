---
title: "Ubuntu 10.04.2上编译ecos工具"
date: 2011-04-17T21:14:00+08:00
draft: false
---

 


主要参考： http://www.ecoscentric.com/devzone/configtool.shtml


次要参考： http://www.cublog.cn/u/2318/showart\_2064983.html


 


1. 下载 wxGTK-2.8.8或更高，编译之：


$ mkdir wx-build


$ cd wx-build


$ /configure --disable-shared --disable-sockets --prefix=


$ make


$ make install


 


$ cd  contrib/src/gizmos         （注：wxbuild/contrib/src/gizmos）
$ make
$ make install
 


2. 编译HOST库及命令行工具（用 新立得软件管理器 安装 tcl8.5 tcl8.5-dev tk8.5 tk8.5-dev）


$ mkdir infra-build


$ cd infra-build


$ /configure --prefix=/opt/ecos-tools --with-tcl-version=8.5 --with-tk-version=8.5


$ make


$ make install


 


3. 编译命令GUI工具


 


1) 编辑 /tools/src/tools/configtool/standalone/wxwin/makefile.gnu


在文件的最上面修改


 


INSTALLDIR=/opt/ecos-tools


WXDIR=


ECOSSRCDIR=/tools/src


 


修改EXTRACPPFLAGS中 -I$(TCLDIR)/include 为 -I/usr/include/tcl8.5


修改EXTRALDFLAGS中 -L$(TCLDIR)/lib 为 -L/usr/lib/tcl8.5


                                             -ltcl 为 -ltcl8.5


 


2) 编译


$ mkdir configtool-build


$ cd configtool-build


$ make -f /tools/src/tools/configtool/standalone/wxwin/makefile.gnu install


 


生成的eCos工具在 /opt/ecos-tools文件夹中


 


