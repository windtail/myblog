---
title: "git 提交中的文件变化"
date: 2012-04-29T23:17:00+08:00
draft: false
---

现在想完成一个操作，即在每次git push之后，自动地根据变化的文件执行一些操作。


这些变化的文件还要分类一下，添加、删除、修改要区分出来。怎么整呢？


拼命查git log命令，没有结果，差一点就想使用 git cat-file命令将这一次和上一次的文件列表进行比较了，


最后发现git其他自带命令，非常好使：


**git diff-tree HEAD HEAD^ --name-status**


输出举例如下：  




M       a.txt  

A       b.txt  

D       c.txt  

  




M表示修改，A表示添加，D表示删除！！  




  

