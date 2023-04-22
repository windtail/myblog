---
title: "debian jessie install note"
date: 2016-12-25T21:58:00+08:00
draft: false
---

Debian支持非常多的硬件，包括arm/mips/ppc/x86，于是想安装个Debian看看，也不想总屈服在canonical的ubuntu下面。


### 光盘镜像太多了


纯社区版的安装总是没有想像得那么好，首先安装镜像居然有3个iso，让我们不想刻光盘的同志怎么办，难道弄3个U盘去。还好有一个网络安装版的，才300M，这个不错，下载下来也挺小了，但是ultroiso写进去不好使，windiskimage写进去也不好使，换到Linux下面写进去就好使了（我是用Centos的UI写进去的，据说cat debian.iso > dev/sdxx也是好使的），mount时type是iso9660


### 没有网卡驱动


U盘顺利启动之后，第一个遇到的问题就是网卡没有驱动（iwlwifi-1000.5.ucode），因为debian安装盘里不包括任何不free的驱动，上网搜了下，去debian官网上下载 firmware-iwlwifi，这是一个deb的包，可以用7zip解压，里面会有需要的ucode文件，本想直接放到安装的这个光盘，不想被mount成iso9660是只读的，于是又找了一个U盘，推荐用fat格式分区，并且据说要放到U盘的根目录或/firmware目录下，我把这个deb文件本身也放到两个目录了，但是仍然不好使，后发现拔掉那个安装光盘，就能正确加载驱动了。


### 无法读光盘


接下来，很不幸的事情发生了，再插上安装U盘，安装时会提示无法读光盘，弄了几次之后，才想到是不是插了光盘加载驱动的事儿。Ctrl+Alt+F2，打开一个shell，发现/cdrom 仍然被mount成/dev/sdc1（安装U盘），但此时的U盘却是 /dev/sdd1，因为加载驱动时又插了一个U盘，怎么插都是 /dev/sdd1，虽然sdc空着，但是就是不用。没办法，只有手动mount下了  
  # umount /cdrom  
  # mount -t iso9660 /dev/sdd1 /cdrom  
终于可以正确读光盘了


### mirror列表中没有阿里云


本人宽带通，访问啥都比较慢，就阿里云的镜像速度还可以，但是选择中国时，就是没有阿里云，只有网易的，但是老出错，第二次选择清华的镜像，但是感觉速度比较慢，第三次仔细看了才发现中国上面还有一个手动设置，把域名填下就好了，即mirrors.aliyun.com，不过下载速度还是很慢。


### 安装选择的程序失败


安装时按默认选择了桌面，下载了1500多个软件包，最后不幸装不上去，就提示失败，没别的提示了。万般无奈跳过了这一步，显示没有安装完，并且由于我电脑上已经安装了Cent-OS，所以我也没有选择安装bootloader（grub或lilo），重启后，进入centos，更新grub2的设置，运行 grub2-mkconfig -o /boot/grub2/grub2.cfg，会自动找到Debian。再次重启后，进入Debian命令行，执行 tasksel install desktop，提示失败，没有任何其他提示，最去apt里搜索，发现其实tasksel对应的apt软件包就是task-desktop，于是尝试 apt install task-desktop，提示依赖有问题，请运行 apt-get -f install ，照着提示做了，再执行 apt install task-desktop，正常安装，重启后就进入Gnome登录界面了！！


### 无法安装中文输入


以前在ubuntu下面用的fcitx挺好，以为Debian应该也用这个，于是安装了 apt install fcitx-table-wubi，然后用im-config手动选择fcitx，无效。最后还是把fcitx卸载掉了，安装了 apt install ibus-table-wubi，然后在配置=>区域和语言，输入法里就可以直接找到五笔了，im-config都不需要运行。


### NetworkManager里无法选择wifi


NetworkManager里无法选择wifi，总是灰色的，但我确实正在用wifi上网，这是怎么回事呢。在Debian NetworkManager包的说明里，发现NetworkManager默认不管理/etc/network/interfaces里指定的网络，打开 /etc/network/interfaces一看，发现wifi热点名称和密码直接以明文的形式写里面了，把wlan0相关的都注释掉，并执行 /etc/init.d/networking restart , /etc/init.d/network-manager restart，就可以在NetworkManager里搜索到其他wifi热点了！


### 如何使能sudo


Debian维护系统时，必须su到root，然后再安装软件等，感觉不是很习惯，还是直接sudo好一点，


安装sudo：apt install sudo


添加当前用户到sudo组：usermod -a -G sudo  （注销后才能生效）


 


暂时只发现这些问题，希望不要再遇到什么别的问题。


