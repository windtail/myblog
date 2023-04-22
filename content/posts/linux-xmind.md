---
title: "Linux XMind"
date: 2018-01-10T15:43:00+08:00
draft: false
---

XMind这个软件好像不错的样子，至少在Windows/Linux/Mac下都可以工作，作为FreeMind的替代品应该是没什么问题（还有一个vym貌似也可以，可能没有XMind好，毕竟XMind有公司在维护）


首先，去官网下载[Linux版](http://www.xmind.net/download/linux/)的，SourceForge的版本已经比较老了。然后解压到主目录中的某个子目录中去，注意，解压文件夹就是运行的文件夹，解压后执行文件夹中的 setup.sh




```
sudo sh setup.sh
```


这会安装依赖库和字体文件，还有一些可选的软件包可以安装




```
sudo apt install libcanberra-gtk-module murrine-themes
```


此时，切换到 XMind\_amd64，双击XMind即可运行！这样运行当然比较原始，我们可为其创建一个应用图标，创建文件 ~/.local/share/applications/xmind.desktop，并将以下内容复制进去（注意修改文件夹位置）




```
[Desktop Entry]
Type=Application
Path=/XMind\_amd64
Exec=/XMind\_amd64/XMind
Name=XMind
Comment=Create mind maps
GenericName=Planning Tool
Icon=/XMind\_amd64/configuration/org.eclipse.osgi/983/0/.cp/icons/xmind.256.png
Categories=Office;
```


注意，上面的Path指定了运行XMind所在的文件夹，这项若是去掉，则会运行出错，提示某个java的类库找不到。


创建了这个应用图标项，就可以在应用程序的办公分类中找到XMind了，enjoy it!


