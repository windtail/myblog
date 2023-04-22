---
title: "cygwin 压缩"
date: 2012-07-03T21:38:00+08:00
draft: false
---

在cygwin中，如果采用 以下命令打包中文文件名的文件，再用winrar打开就是乱码  

$ tar cjf a.tar.bz2 中文名文件


而采用7z就不会有问题，命令行如下：


$ 7z u a.7z 中文名文件


注：7z在p7z软件包中  




