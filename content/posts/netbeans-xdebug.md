---
title: "netbeans xdebug"
date: 2013-06-25T21:08:00+08:00
draft: false
---

xdebug配置
--------


装了wamp后，xdebug默认就安装好了，为了能够用netbeans远程调试，配置文件里得加几句




```
[xdebug]
xdebug.remote\_enable = on
xdebug.remote\_handler=dbgp
xdebug.remote\_host=localhost
xdebug.remote\_port=9000
```


 Netbeans配置
-----------


工具选项


* 常规=>Web浏览器=>Firefox
* PHP=>常规=>PHP 5解释器=>\bin\php\php5.x.xx\php.exe
* PHP=>调试：这里默认就行了，默认就是9000号端口


调试
--


打开要调试的文件，选择“调试=>调试文件”菜单即可


