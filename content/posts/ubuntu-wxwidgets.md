---
title: "Ubuntu wxWidgets"
date: 2011-08-03T23:29:00+08:00
draft: false
---

1. 用新得立安装如下软件包：  




＃ codeblocks  

＃ codeblocks-dbg  

＃ wxformbuilder  

＃ libwxgtk2.8-dev  

＃ libwxgtk2.8-dbg  




安装完后，主菜单的 编程菜单中会有 codeblocks和wxformbuilder


  




2. 配置codeblocks  




# 打开codeblock -> settings->global variables  

# current variable标签后面点击new按钮，出来的框里填写wx。  

# 然后builtin fields下面  

   base         /usr  

   include    /usr/include/wx-2.8  

   lib             /usr/lib


配置完成后，重新启动codeblocks


  




3. 新建Hello World程序


New -> WxWidgets Project


然后按提示填就可以了，将会生成一个工程，选择工具条中的编译并执行，就会弹出一个窗口，成功！！


参考：http://forum.ubuntu.com.cn/viewtopic.php?f=70&t=246620&start=0  




