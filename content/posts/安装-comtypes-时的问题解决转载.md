---
title: "安装 comtypes 时的问题解决（转载）"
date: 2013-11-21T22:49:00+08:00
draft: false
---

**转载地址：<http://idaemon.net/post-213.html>**


**Reference:**   
<http://www.dev-club.net/xiangxixinxi/22010072906013419/201012250940122.html> 


下载地址：   
<http://sourceforge.net/projects/comtypes/files/>


不建议使用win32安装包，遇到问题无法调试和修改，比如我就遇到了这个安装包无法检测到python安装目录的问题，可能是我用的x64或者是python2.7有关吧。


总之，下载zip包，解压，使用正常的安装方式：


python setup.py install


如果遇到下列报错提示：


Traceback (most recent call last):      
File "\comtypes\setup.py", line 42, in       
from distutils.core import setup, Command, DistutilsOptionError      
ImportError: cannot import name DistutilsOptionError 


则打开comtypes的setup.py，找到（其实就是42行啦）下列内容:


from distutils.core import setup, Command, DistutilsOptionError


替换为：   
from distutils.core import setup, Command   
from distutils.errors import DistutilsOptionError


保存后重新 python setup.py install 即可。


