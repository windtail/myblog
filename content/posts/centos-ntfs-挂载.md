---
title: "CentOS  NTFS 挂载"
date: 2010-10-10T22:38:00+08:00
draft: false
---

 


话说我安装完CentOS后，还有许多文档文件在windows的NTFS分区，我不能不用啊，怎么挂载NTFS呢。有个哥们说了：


http://whr25.blog.sohu.com/117853982.html 不过照着他说的做，yum说网易里只有fuse，其他的都没有，这可怎么办，这时又有人说要换个源，这个我可不想干，好不容易才连上网易，怎么能换了，正当这时，一位同学说http://hi.baidu.com/taodoor/blog/item/318963d9087ead2910df9b68.html，咱们linux下最原始的安装方法可不是yum啊，嗯，确实是这么回事，咱们还可以下载软件源码make下嘛！


 


到http://www.tuxera.com/community/ntfs-3g-download/下载源码，照说这页说的做，就可以了（注意：前面的fuse还是要装的）


 


至于分区的信息，可以用 fdisk -l查询，上网一搜，还可以用sfdisk parted等等查分区信息。


