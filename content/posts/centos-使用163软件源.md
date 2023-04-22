---
title: "CentOS 使用163软件源"
date: 2010-10-10T21:53:00+08:00
draft: false
---

以前都用Fedora，不过这两天发现Fedora实在是更新太快了，其他软件都跟不上节奏。这才想起来CentOS这个东西，大家都 说他稳定，我感觉也是，你看人最新的5.5版的内核才2.6.18，Fedora早就2.6.32了。


 


言归正传，虽然CentOS稳定，但是软件还是要装的，有时也得更新，软件源就不得不选国内的了，163当然是首选。今天发现网易做事可真是到位可以直接在网易的网站上下载到写好的repo文件，http://mirrors.163.com/.help/CentOS-Base-163.repo


下载后放到/etc/yum.repos.d/中，将其他的源的文件名后面都直接加个.bak让他们失效，另外把CentOS-Base-163.repo文件中的mirrorlist行注释掉（前面加#），这样保证总是从网易下载。完成后使用两个命令：


  yum clean metadata  //清除以前的缓存


  yum makecache          //重新建立缓存


 


不过国内还有搜狐的镜像，所以也可以自己做个local mirrorlist，不过我没有试过，这个网页有介绍Fedora的，http://fedoranews.org/tchung/yum-mirrorlist/


 


网易每个系统后面都有个使用帮助的链接，照着上面做就好使。


