---
title: "如何上传package到pypi"
date: 2018-05-23T16:35:00+08:00
draft: false
---

首先访问 [pypi](https://pypi.python.org) 创建一个帐号，并且需要验证一个邮箱，注意网易163邮箱收不到验证的邮件。


安装上传工具
------




```
pip install --user twine
```


执行上传命令
------




```
python setup.py sdist bdist\_wheel
twine upload dist/\*
```


注意，dist/ 下面应该只能wheel和代码压缩包，如果还有别的文件，建议删除之


更新代码后再次上传
---------


更新代码后再次上传必须要更新版本号，pypi不允许上传同名文件，即使用你把之前的删除了，它也不会再让你上传了。也就是说即使你是修正BUG，也要更新版本号，否则万一别人的代码依赖你这个BUG，就会不好使了。


 


