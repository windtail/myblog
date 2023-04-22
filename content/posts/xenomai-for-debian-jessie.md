---
title: "Xenomai for Debian Jessie"
date: 2016-12-30T00:31:00+08:00
draft: false
---

### 安装内核源码包


apt install linux-source-3.16


安装其他编译需要的工具： apt install build-essential libc-dev libc6-dev pkg-config ncurses-dev


安装好的内核源码和config文件在 /usr/src中，linux-source-3.16.tar.xz是源码压缩包（找个地方解压下，下面称解压的linux源码根目录为 $linux-tree），而linux-config-3.16中有需要的config文件，找一个和 uname -r 输出相似的config，对于32位的Debian来说应该是 config.i386\_none\_686-pae.xz，解压之后重命名为 .config放到 $linux-tree中去，现在可以编译下内核试试了


* make menuconfig （输入 / 开始搜索，然后输入localversion，按1选择本地版本，输入一个自己能识别的后缀）
* make bzImage modules
* sudo make modules\_install
* sudo make install


这样编译好的内核就自动安装到 /boot/ 目录下去了，而模块在 /lib/modules/3.16.36xxx 下面了


简单运行下 grub2-mkconfig -o /boot/grub2/grub.cfg 就可以将新编译的内核放到开机的grub菜单中去了（赶紧试试吧）


### 下载Xenomai和I-pipe


https://xenomai.org/downloads/xenomai/stable/latest/ 我下载的是 3.0.3 版本，解压，后面称xenomai源码根目录为 $xeno-tree


还要下载对应版本的 I-pipe 补丁，我下载的是这个 https://xenomai.org/downloads/ipipe/v3.x/x86/older/ipipe-core-3.16-x86-3.patch （后面我发现好像有未发布的对应 3.16.36版本的i-pipe补丁，不过我没有再试）


安装编译工具：apt install autoconf automake libltdl-dev


（注意：必须安装 libltdl-dev，否则bootstrap都通不过，参见 https://xenomai.org/faq/ ）


### Patch内核


虽然Xenomai3也可以在正常的linux下运行，但那当然不是我们想要的，我们还是要双内核架构的 cobalt，首先就要把patch打内核上去，由于我们没有对应到内核小版本的补丁，所以这一步还需要手动来调整一下


* cd $linux-tree
* patch -p1 < ipipe-core-3.16-x86-3.patch


会出现一些失败，还有一些偏移后成功的项，根据我这次的经验，直接显示成功的就不用管了，fuzz后成功的要注意下，偏移太多了要重点检查，很有可能就错了，如果失败了，会生成一个 .rej 文件，所以在 $linux-tree 目录，执行 find . -name \*.rej 就能找到这些失败的位置了。每个 @@表示一个patch块的开始，而同行最后一个@@后面表示当前的context（即在修改块上面的代码，可以先搜索到这个位置，再看），下面没有＋或－的也是改动的未改变的上下文，－表示删除 ＋表示添加，仔细看看，一般失败的情况也很好判断出应该如何修正，极少会遇到函数更名的情况，需要注意一下，建议大家手动来一遍。


把所有的rej都搞定了之后，再根据xenomai的说明来准备内核，即执行 scripts/prepare-kernel.sh --linux=$linux-tree --arch=x86


OK下面就是配置内核了， make menuconfig，你可能会看到 Xenomai下面有一堆warning，根据这个网页来配置就可以了 https://xenomai.org/2014/06/configuring-for-x86-based-dual-kernels/，即


* CONFIG\_CPU\_FREQ – Disable
* CONFIG\_APM – Disable
* CONFIG\_ACPI\_PROCESSOR – Disable
* CONFIG\_CPU\_IDLE – Disable
* CONFIG\_INTEL\_IDLE – Disable
* CONFIG\_INPUT\_PCSPKR – Disable
* MIGRATION／COMPACTION －Disable


如果还有其他的warning，请自行尝试把warning消除，多看看help，有一些项被其他项所选择，不能直接改变，要先改选择了它的项。另外注意ACPI不能全部都不选择，那样会启不了机，或者启机特别慢（我的是PCI IRQ错误，并且USB都不好使），ACPI只把 PROCESSOR选掉。


好了，重新编译内核吧 make bzImage modules && sudo make modules\_install && sudo make install


重新启动，不出意外的话就可以启机了，启动后　dmesg | grep Xenomai，应该会有一些输出，否则就是有问题了。另外，有一个smi的问题，这个需要修改grub的启动参数，给内核参数后面加上　xenomai.smi=enabled，不过这项可能会导致硬件损坏，比如CPU过热烧掉了（看这里　https://xenomai.org/2014/06/configuring-for-x86-based-dual-kernels/dealing-with-x86-smi-troubles　），自己看着办吧。


### 编译xenomai库


* cd $xeno-tree
* scripts/bootstrap
* mkdir build
* cd build
* ../configure --with-core=cobalt --enable-smp --enable-pshared  （基本上大家应该都是多核处理器，默认内核都打开了SMP）
* make
* sudo make install


安装完的库在 /usr/xenomai 文件夹中，执行 sudo /usr/xenomai/bin/xeno-test 即可，注意必须用root才能运行测试，另外这个测试默认不会自己结束，必须 Ctrl+C结束，　看看　/usr/xenomai/bin/latency --help，就知道 latency　测试的参数了。


PS：我的T410i本子跑个实时任务还是不错的，50us轮循周期，最大latency为11us多，还可以！


