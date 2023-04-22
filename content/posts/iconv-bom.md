---
title: "iconv bom"
date: 2012-08-23T00:05:00+08:00
draft: false
---

iconv是一个经常用来转换编码的工具，非常好用。但是今天晚上用它来将utf-8转换成utf-16时，发现它总是会自动在前面插入FEFF的BOM，转换的命令行如下：




```
iconv -f utf-8 -t utf-16 
```


而我是不需要这个BOM的，当然我可以用外部程序再进行转换一次，去掉前面的BOM，但是难道iconv真就没有考虑到吗？


虽然官方文档貌似没有关于这方面的说明，但是google了下，答案果然是否定的，参见[这个帖子](http://superuser.com/questions/381056/iconv-generating-utf-16-with-bom)，也就是说命令行换成这样就行了




```
iconv -f utf-8 -t utf-16le 
```


