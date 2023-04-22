---
title: "X without window manager（无桌面的X）"
date: 2021-03-13T23:46:00+08:00
draft: false
---

当我们搞嵌入式的时候，我们经常不需要桌面，开机就运行咱们的程序。这个在上位机（x86上）可以搞吗，当然可以，并且非常地简单。


终端中直接运行Qt程序
-----------


我们写一个Qt的程序，其实它就是一个X的client，我们只需要一个Xserver就好了，那Xserver怎么启动呢，很容易，运行 xinit 就可以了，甚至不需要运行startx，据说startx脚本各发行版都不一样，所以直接运行xinit比较靠谱。xinit不带参数时，会运行~/.xinitrc，如果xinit带一个路径参数（区分路径参数和非路径参数就是参数中是否带slash，即/），xinit会视路径参数为一个client程序，启动Xserver，然后运行这个路径参数指向的程序。如果xinit带的参数不是一个路径，那么xinit还是会去启动~/.xinitrc并把参数传给这个脚本。


我们来试试，首先我们要开机进入终端模式（不要启动Xserver）




```
sudo systemctl set-default multi-user.target
```


重启后，在终端中登录，执行




```
xinit /usr/bin/xterm
```


就会启动xterm（如果你安装了的话，没安装的话，apt install xterm下）


如果我们建一个~/.xinitrc，并把以下行放进去




```
exec /usr/bin/xterm
```


然后直接执行xinit，就会上面的代码一样。


然后，我们如果把xinit放到shell初始化脚本的最后，不就可以登录直接运行某个Qt程序了么，但是我想自动登录可以吗，貌似命令行有点复杂。参考


<https://unix.stackexchange.com/questions/401759/automatically-login-on-debian-9-2-1-command-line>


使用SDDM自动登录，开机自动运行指定程序
---------------------


好了，我选择简单一点，我们安装下sddm，并将其选择为使用的display manager，开机init最后就会运行到display manager，这个时候已经有界面了，所以Xserver实际已经启动了。SDDM负责登录认证，认证之后会来启动一个session，这个session程序们默认在 /usr/share/xsessions/ 中找，看看这个文件夹中的其他文件，我们会发现这个session其实蛮简单的，我们也来写一个myxterm.desktop放这个文件夹里面




```
[Desktop Entry]
Name=MyXterm
Comment=MyXterm experiment
Type=Application
Exec=/usr/bin/xterm
```


好了，是时候恢复成graphical了，




```
sudo systemctl set-default graphical.target
```


重启就会看到左上角，选择框中多了一项咱们的MyXterm，试试效果吧


说好的自动登录呢，man sddm.conf 看看


我们创建一个 /etc/sddm.conf，把下面的行保存下




```
[Autologin]
# Whether sddm should automatically log back into sessions when they exit
Relogin=true

# Name of session file for autologin session (if empty try last logged in)
Session=myxterm

# Username for autologin session
User=xxx
```


用户名称记得改下，试试效果吧，自动登录，并且退出还会自动重启。


enjoy it!


 


参考：


1. https://en.wikibooks.org/wiki/Guide\_to\_X11/Starting\_Sessions


2. https://github.com/sddm/sddm


3. man sddm


4. man sddm.conf


 


