---
title: "rtems 4.11 工具链"
date: 2016-07-21T22:23:00+08:00
draft: false
---

4年前，曾经把rtems4.10移植到atmel 9263上，要不是当时移植的git仓库还在的话，真是不相信自己居然还干过这事。所以这次再捡起的时候，要记录一下。还是从编译器开始。


首先打开 <http://docs.rtems.org/rsb/> 这里说了很多为什么要从源代码编译的好处。大概就是说我们的产品要维护很多年，我们要把编译环境保留下来，但是我们不能总是老的操作系统开发啊，比如现在都出centos7，咱不能还用centos6吧，也不能在公司保存一堆老操作系统的机器，就为了保护编译环境吧，再说还得调试。


按照说明文档步骤来：


1. git clone git://git.rtems.org/rtems-source-builder.git
2. cd rtems-source-builder
3. **cd rtems** （这一步非常重要，否则后面列表时，会少很多bset）
4. ../source-builder/sb-check  （这会告诉你缺少，照着安装就行了）
5. ../source-builder/sb-set-builder --list-bsets （会列出一大堆可用的编译配置，咱们这里只关心 **4.11/rtems-arm.bset** ）
6. ../source-builder/sb-set-builder --log=build-arm.txt --prefix=**$HOME/development/rtems/4.11** 4.11/rtems-arm （其中prefix=的就是最后编译完工具链的位置）


可以去喝茶了，然后回来……（咱们和老外还是有差别的，人家回来就编译好了，我们回来很可能发现是下载错误……），所以第6步，咱得先--dry-run下：


* ../source-builder/sb-set-builder --log=build-arm.txt --prefix=**$HOME/development/rtems/4.11** 4.11/rtems-arm --dry-run | egrep '(http|ftp|https)://' （这样我们就知道要下载哪些了，并且下载到的位置也知道，大部分都放到 source目录下，有几个patch放到 pathes目录下，OK用咱们的各种下载神器下载吧，然后再执行第6步，这下真的可以喝茶去了！）


好，没啥问题就编译完了，出错就别找我了，我没出错……（我这是 CentOS 7）


最后就是把编译完的加到path环境变量里去，PATH=$HOME/development/rtems/4.11:$PATH


