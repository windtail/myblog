---
title: "Debian 为nginx增加第三方模块"
date: 2018-05-25T15:16:00+08:00
draft: false
---

为nginx增加第三方模块需要重新编译nginx的，但是debian在安装nginx的时候做了很多事情，比如systemd，/etc/nginx/里的各种文件，所以我们最好在debian源代码包的基础上改一改。


添加nginx官方apt仓库
--------------


参考 [nginx官方文档](http://nginx.org/en/linux_packages.html)，下载 nginx的[key](http://nginx.org/keys/nginx_signing.key)到，并添加到系统




```
sudo apt-key add nginx_signing.key
```


在 `/etc/apt/sources.list` 中添加如下两项，注意 deb-src 非常重要




```
deb http://nginx.org/packages/debian/ stretch nginx
deb-src http://nginx.org/packages/debian/ stretch nginx
```


如果你不是stretch，请自己修改为 jessise 等。完成之后，执行




```
sudo apt-get update
```


源码编译
----


参考[这篇](https://serversforhackers.com/c/compiling-third-party-modules-into-nginx)文章，安装编译工具和源代码




```
cd
mkdir nginx-build
cd nginx-build
sudo apt-get install -y dpkg-dev
sudo apt-get source nginx
sudo apt-get build-dep nginx
```


 


打开 nginx-build/nginx-/debian/rules 文件，找到 config.status.nginx 下面的 CFLAGS，在靠后的位置添加你要编译的模块，例如




```
--add-module=/path/to/your/module
```


 


修改好了之后，在 nginx-build/nginx-目录下面执行




```
sudo dpkg-buildpackage -b
```


 


编译完成之后，会在 nginx-build 目录下生成 deb包，如 nginx\_1.14.0-1~stretch\_amd64.deb


安装
--


如果在debian的main仓库中安装了 nginx-full，请先卸载之。然后执行




```
sudo dpkg -i nginx_1.14.0-1~stretch_amd64.deb  # 根据你自己生成的deb文件修改
```


 


可以执行 sudo nginx -V 来查看是不是真的包含了你的模块


