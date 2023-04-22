---
title: "Windows xp下建立git服务器及bug追踪"
date: 2012-01-04T23:08:00+08:00
draft: false
---

1      SSH服务器
=============


1.1    安装open-ssh软件包
--------------------


在Ubuntu上建立SSH服务器是非常简单的，但是XP上就要费点劲了。首先，安装Cygwin。好在网易提供了Cygwin的镜像，所以这一步现在变得异常地简单。


1.        从Cygwin的官方网站[http://Cygwin.com](http://cygwin.com/)上下载setup.exe


2.        安装时选择[http://mirrors.163.com](http://mirrors.163.com/)，下载速度特别地快


3.        选择软件包OpenSSH，安装直到完毕


1.2    安装sshd服务
---------------


1.        安装完毕后，将Cygwin安装文件夹里的bin文件夹“C:\Cygwin\bin”放Path环境变量中


2.        双击桌面上的Cygwin图标打开控制台，输入 “ssh-host-config -y” 将sshd注册为系统服务，并设置成为自动启动


3.        手动控制ssh服务启动（也可以重启电脑）“net start sshd”（关闭为“net stop sshd”）


4.        配置防火墙，打开ssh的端口，默认是22（tcp）


1.3    配置sshd服务器
----------------


用UltraEdit或写字板打开配置文件C:\Cygwin\etc\sshd\_config


1.        #PermitRootLogin yes => PermitRootLoginno                           #禁止root登录


2.        #PasswordAuthentication yes=> PasswordAuthenticationno       #仅使用密钥登录


3.        #Protocol 2,1 => Protocol 2                                                     #只允许SSH2方式


1.4    为用户添加公钥
--------------


### 1.4.1  生成公钥和私钥


1.        从[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/%7Esgtatham/putty/download.html)处，下载putty.zip


2.        解压后运行puttygen.exe


3.        在最下面Parameters里选择SSH-2 RSA，Number of bits in a generated key填1024（默认就是这样）


4.        点击Generate，然后鼠标随机移动，直到生成完毕


5.        复制公钥（最上面的框里的，以ssh-rsa打头）到文本文件中，并在存储为authorized\_keys文件


6.        点击Save private key保存私钥文件


注意：千万不要直接点Save public key来保存公钥，那样会生成很多无用的字符串，导致无法登录。


### 1.4.2  配置公钥


1.        切换到用户的主目录：cd ~


2.        新建.ssh文件夹：mkdir .ssh


3.        将刚才保存的公钥文件authorized\_keys拷贝到.ssh文件夹中


4.        修改authorized\_keys的属性为600：chmod 600 authorized\_keys


注意：.ssh文件夹的属性group和other也不能具有写权限，否则无法登录。


注意：每一个服务器用户可以有多个公钥和私钥对


### 1.4.3  尝试连接sshd服务器


运行putty.zip解压出来的putty.exe


1.        Session里Host name填写安装了sshd服务器的ip


2.        Connection->Data里的Auto-loginusername填你刚才配置的公钥的用户名


3.        Connection->SSH->Auth里的Private Keyfile for authentication选择刚才生成的私钥文件


4.        点击Open，应该就能连接上了


2      安装git
============


2.1    服务器端
-----------


1.        重新运行Cygwin的setup.exe，安装git


2.        初始化一个bare的仓库，比如：在/opt/git/test.git文件夹中运行 git --bare init


3.        保证你设置的用户名有权限读写test.git


2.2    客户端
----------


1.        安装msysgit和tortoisegit最新版（都按默认方式安装）


2.        tortoisegit找一个空文件夹，右键点击clone


3.        url里输入@:/opt/git/test.git


4.        在下面选中Load Putty Key，选择生成的私钥文件，然后点击OK，无意外的话，应该是可以成功clone的


5.        添加一个文件，看看能不能push，没有意外，应该是可以的


3      通过http拷贝仓库
=================


3.1    安装apache
---------------


1.        重新运行Cygwin的setup.exe


2.        安装apache httpd2


3.        添加用户环境变量CYGWIN=server（不知道为什么添加，不添加就不好使）


4.        重新打开Cygwin命令行


5.        执行cygserver-config，注册cygserver为windows系统服务


6.        手动启动cygserver服务：net start cygserver


7.        运行apache服务器：/usr/sbin/apachectl2 start


8.        打开浏览器访问http://localhost，It works!


3.2    配置git仓库
--------------


1.        将仓库中的hooks文件夹中的pre-update.sample重命名为pre-update


2.        添加执行属性：chmod a+x hooks/pre-update


注意：这个配置使得每次push之后，都会更新服务器的info，保证通过http，clone是正确的。


3.3    配置apache
---------------


1.        打开/etc/apache2/http.conf文件


2.        搜索httpd-vhosts.conf，将#Include /etc/apache2/extra/httpd-vhosts.conf这一行前面的#去掉


3.        打开/etc/apache2/extra/httpd-vhosts.conf文件


4.        在结尾添加如下内容


  




 



         ServerAdmin


         ServerNamegit.gitserver


         DocumentRoot/opt/git


         


                   Orderallow,deny


                   allowfrom all


         



 


5.        重新启动apache服务器


a)        /usr/sbin/apachectl2 stop


b)        /usr/sbin/apachectl2 start


3.4    客户端clone方法
-----------------


1.        配置DNS服务器或直接修改客户端的hosts文件，将服务器的ip指向git.gitserver


2.        在客户端运行 git clone http://git.gitserver/test.git 不出意外的话应该就能匿名clone了


4      建立gitweb服务器
==================


即通过网页访问git仓库。


4.1    下载git源码
--------------


1.        重新运行Cygwin的setup.exe，


a)        安装git的source file，安装的源文件在/usr/src中


b)        安装gcc4和make


2.        解压git源文件，进入解压的文件夹


3.        make GITWEB\_PROJECTROOT=”/opt/git”prefix=/usr gitweb/gitweb.cgi


4.        cp –Rf gitweb /srv/www/


注意：GITWEB\_PROJECTROOT指定git仓库集文件夹


注意：/srv/www/是apache httpd2建立的文件夹，也可以放其他文件夹


4.2    配置apache
---------------


1.        打开/etc/apache2/extra/httpd-vhosts.conf文件


2.        在结尾添加如下内容


 



         ServerNameweb.gitserver


         DocumentRoot/srv/www/gitweb


         


                   OptionsExecCGI +FollowSymLinks +SymLinksIfOwnerMatch


                   AllowOverrideAll


                   orderallow,deny


                   Allowfrom all


                   AddHandlercgi-script cgi


                   DirectoryIndexgitweb.cgi


         



 


3.        重新启动apache服务器


4.3    客户端访问方法
--------------


1.        配置DNS或直接修改客户端的hosts文件，使得web.gitserver指向服务器


2.        在浏览器中输入[http://web.gitserver](http://web.gitserver/)


3.        不出意外，就可以直接访问git仓库了


5      安装redmine
================


注：安装完后，我发现官网上提供了一个链接，做了一个windows下的安装包，不知道好不好使。链接<http://bitnami.org/stack/redmine>如果好使的话，下面的就不用看了。


注：今天试了试bitnami，确实挺好使的，一键装好apache，mysql，ruby/rails/rake，redmine，自动配置好！只要注意几个问题：


1. 安装时一定要关闭杀毒软件和安全软件，比如360等
2. 所有填的东西都不要有中文，比如人名，否则不能正确生成配置文件
3. 注意以前没有安装冲突的东西，比如imagemagick（如果最后都装好了，但是就是不能启动redmine，看看log文件）


5.1    安装ruby/rails/rack等
-------------------------


1.        在rubyforge中下载并安装rubyinstaller-1.8.7-p357.exe（不要安装新版，新版不好使）


2.        在开始菜单中，选择Ruby 1.8.7-p357-> Start Command Prompt with Ruby


3.        gem install rails -v=2.3.11


4.        gem install rack -v=1.1.1


5.        gem install mysql


6.        下载官网安装说明中libmysql.dll <http://instantrails.rubyforge.org/svn/trunk/InstantRails-win/InstantRails/mysql/bin/libmySQL.dll>拷贝到ruby的bin文件夹


7.        gem update --system 1.6.2 （否则后面的db:migrate会报错）


5.2    下载redmine并解压
-------------------


1.        从官网链接下载redmine 1.3.0 <http://rubyforge.org/frs/?group_id=1850>


5.3    安装并配置mysql
-----------------


1.        下载mysql的win32版，并安装（我安装的是mysql-5.5.19-win32.msi），记得钩选将mysql的执行文件路径加入到Path环境变量


2.        打开cmd，执行mysql -u root -p，并输入密码，然后执行以下3个sql语句


a)        create database redminecharacter set utf8;


b)        create user'redmine'@'localhost' identified by 'my\_password';


c)        grant all privileges onredmine.\* to 'redmine'@'localhost';


5.4    配置redmine
----------------


1.        进入redmine解压的文件夹，重命名config/database.yml.example为config/database.yml，并修改production下的内容为


 


production:


  adapter: mysql


  database: redmine


  host: localhost


  username: redmine


  password:my\_password


 


2.        在redmine文件夹执行


a)        rake generate\_session\_store


b)        set RAILS\_ENV=production


c)        rake db:migrate


d)        rake redmine:load\_default\_data


3.        测试：


a)        ruby script/server webrick -eproduction


b)        在浏览器中访问[http://localhost:3000](http://localhost:3000/)


c)        登录管理员登录：用户名 admin 密码 admin


5.5    mongrel
--------------


webrick服务器只能用来做测试，发布时可以采用mongrel服务器或apache服务器，不过apache对应的module太难整了，mongrel比较好配置。


### 5.5.1  安装mongrel


1.        安装mongrel：gem install mongrel


2.        安装mongrel\_service：gem install mongrel\_service


### 5.5.2  下载mongrel bug补丁


rails和mongrel版本有点不兼容导致的问题。


1.        下载<https://gist.github.com/raw/826692/cb0dcf784c30e6a6d00c631f350de99ab99e389d/mongrel.rb>


2.        运行 rails --version 看看rails到底是什么版本，我的居然是2.3.14，我明明安装的是2.3.11……


3.        打开mongrel.rb文件，第一行if后面一看就知道是个数组，在最后加上你的rails的版本，比如[,’2.3.14’]，保存之


4.        将mongrel.rb文件保存到redmine文件夹中的config\initializers\子文件夹中


### 5.5.3  注册mongrel服务


1.        注册服务：mongrel\_rails service::install -N redmine -c -p 3000 -e production


2.        打开控制面板->管理工具->服务，找到redmine，设置为“自动”，使得redmine服务开机启动


 


注：


1.        手动启动服务：mongrel\_rails service::start -N redmine


2.        手动停止服务：mongrel\_rails service:: stop -N redmine


3.        删除服务：mongrel\_rails service::remove -N redmine


### 5.5.4  mongrel和apache同时运行


据说可以用apache的mod\_proxy模块，将apache的某个folder映射到mongrel所在的端口，不过不太会整，放弃之。


用到的资源
=====


为了方便大家，把安装时用到的文件上传了，两个包：


<http://download.csdn.net/detail/windtailljj/3999049>


<http://download.csdn.net/detail/windtailljj/3999070>  




