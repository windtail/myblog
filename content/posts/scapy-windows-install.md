---
title: "scapy windows install"
date: 2016-03-17T21:10:00+08:00
draft: false
---

最近有点扫描网络的需求，都说scapy好，但是安装是个事（当然指的是windows安装）  
有个[scapy3k](https://github.com/phaethon/scapy)，支持python3，可惜需要powershell，也就是说windows xp是没有戏了。


网上说最好python2.6，但这年头，谁还能忍python2.6啊，连centos都已经是python2.7了，scapy有几个重要的依赖，


* WinPcap，这个好办，[官方下载](http://www.winpcap.org/install/bin/WinPcap_4_1_3.exe)一个，或者安装上最新wireshark，就有了
* pypcap，可以从pypi上下载[源代码](https://pypi.python.org/packages/source/p/pypcap/pypcap-1.1.4.tar.gz#md5=e2ded33e75d94e4635798a2b2f4aaea1)编译，需要安装 vs2008，还需要winpcap的开发包，在这页有[下载](http://www.winpcap.org/devel.htm) 当前是[4.1.2](http://www.winpcap.org/install/bin/WpdPack_4_1_2.zip)  ，下载后解压，然后修改pypcap的setup.py中dirs变量，dirs=['path/to/wpdpack'] ，修改后 python setup.py install 即可
* libdnet，这个实在是自己不好编译，文档又太少，据说要同时安装cygwin和mingw还有vs2008才能搞定，反正我是没有搞定，于是从[stackoverflow](http://stackoverflow.com/questions/5447461/running-scapy-on-windows-with-python-2-7)里搞到一个[下载链接](http://dirk-loss.de/scapy/dnet-1.12.win32-py2.7.exe)
* numpy这个好安装，就不说了


最后是下载scapy源代码，然后执行python setup.py install 即可


 


