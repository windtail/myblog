---
title: "how make none-bare git repo to bare"
date: 2011-05-04T19:54:00+08:00
draft: false
---

 


在云存储的时代，各种网盘都来了。


自己没事写点代码时，用用git，利用网盘的自动同步功能，便可以自己架一个私有的准git server了，


原理很简单的，把git repo放网盘里就可以了。


 


不过，这个时候我们需要将原来不是bare的repo，弄成bare的才完美。


摘自[GIT FAQ](https://git.wiki.kernel.org/index.php/GitFaq#How_do_I_make_existing_non-bare_repository_bare.3F)
，自己试了一下。


 


我的环境： Windows XP + msysgit


步骤： 


  \* 假设原来的repo文件夹为 F:/repo


  \* 在网盘的目录下新建一个文件夹，例如 test


  \* 右键文件夹，选择 Git Bash


  \* git clone --bare -l /f/repo ./


经过以上步骤，test就是一个裸的仓库，以后要用它就可以这样：


  \* 切换到目标文件夹


  \* git clone <网盘目录>/test


  OK


 


 


20110606 注：忽然发现其实 TortoiseGit 在Clone的时候就有一个选项可以 clone成一个bare的repo，呵呵


