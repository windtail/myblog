---
title: "Debian Epson L455 打印机"
date: 2018-02-23T23:10:00+08:00
draft: false
---

要使用打印机必须要在本机（或局域网其他机器）上安装打印服务，L455是网络打印机，需要IPP协议，而mDNS-scan和avahi-utils是用来发现网络打印机的服务，由于我是摸索着安装的，没太研究他们之前的依赖关系。说下我的安装步骤




```
sudo apt install mdns-scan
```


此时运行mdns-scan可以发现我的打印机在哪个IP上




```
sudo apt install avahi-utils
```


此时运行avahi-browser -art可以发现打印机的IP和端口




```
sudo apt install cups-ipp-utils
```


此时运行ippfind可发现打印的ipp的uri，打开打印设置（搜索printer），发现没有打印服务，于是又安装




```
sudo apt install cups
```


在打印设置界面右上角点[解锁]，然后选择添加打印机，网络中会自动扫描到L455，选择之，这里会要求选择驱动，各种试之后不好使，后google之，发现还需要安装




```
sudo apt install printer-driver-escpr
```


注：打印设置界面的对话框在我这里特别大，无法点到按钮的，方法是按win键看看缩小的窗口，看看按钮的快捷键，比如forward是Alt+f，而Apply是Alt+a


