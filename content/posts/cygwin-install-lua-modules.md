---
title: "cygwin install lua modules"
date: 2012-07-29T23:43:00+08:00
draft: false
---

写一篇博客纪念我今天的辛苦工作，虽然最后也不完美，但是这一点工作也许能给大家一点帮助，省得大家再重复走路了。


  




最近用总用Lua和Cygwin，但Lua用的是LuaForWindows，因此不是原生态的cygwin的东西，其实我很想用cygwin中的Lua，但是cygwin中的lua没有模块啊，都要自己整，想想都觉得费劲。总希望有个人来做这件事，但是今天我终于忍不了了。整了一天，也没有把tecgraf的IUP/IM/CD给整上去，太菜了，没办法，先把整了的东西放上来吧。


  




#### 安装Cygwin


安装时必须保证安装如下模块：


lua autoconf automake autobuild gcc4 gcc4-g++ wget curl unzip make cmake git libzzip-devel libxml2-devel diff patch libgd-devel libgd2 pkg-config libcurl4 libcurl-devel libsqlite3-devel  




#### 安装Lua的模块们


安装了如下模块：


luarocks luazip luaxml lunit luasocket luafilesystem lualogging luadoc md5 date bit32 lpeg luacurl luasql-sqlite3 stdlib lbase64 lpack luaprofiler luagd luacom  




安装的步骤有很多，写了3个脚本，生成了一个patch，把这它们下载下来，放到一个文件夹中，切换到那个文件夹，执行 ./install.sh 即可


  




**install.sh**




```
#!/bin/sh

START_DIR=$PWD # gd.patch MUST be here
LUA_CLIB_DIR=/usr/local/lib/lua/5.1
LUA_LLIB_DIR=/usr/local/share/lua/5.1
RUN_DIR=~/tmp/luainstall

# run in ~/tmp
mkdir -p $RUN_DIR
cd $RUN_DIR

# install luarocks
echo "####################### Installing luarocks #################"
wget "http://luarocks.org/releases/luarocks-2.0.10.tar.gz"
tar xzf luarocks-2.0.10.tar.gz
cd luarocks-2.0.10
./configure
make
make install
cd $RUN_DIR

# install lua modules by luarocks
MODULES="luazip luaxml lunit luasocket luafilesystem lualogging luadoc md5 date bit32 lpeg luacurl luasql-sqlite3 stdlib lbase64 lpack luaprofiler"

for mod in $MODULES ; do
	echo "############### Installing $mod #####################"
	luarocks install $mod
done

# install lua modules that cannot by luarocks
echo "##################### Installing luagd #######################"
wget "http://files.luaforge.net/releases/lua-gd/lua-gd/lua-gd-2.0.33r2forLua5.1/lua-gd-2.0.33r2.tar.gz"
tar xzf lua-gd-2.0.33r2.tar.gz
cd lua-gd-2.0.33r2
patch < $START_DIR/gd.patch # the script must run at the $RUNDIR
make
cp -f gd.so $LUA_CLIB_DIR/
cd $RUN_DIR

# install luacom
echo "####################### Installing luacom #####################"
rm -rf luacom #enable run more than once
git clone https://github.com/davidm/luacom.git
cd luacom
mkdir -p build
cd build
cmake ..
make
cp luacom.dll $LUA_CLIB_DIR/
cd $RUN_DIR

# change the default lua path
echo "###################### Changing the default lua path/cpath ######################"
# enable run more than once
CHANGED=$(cat ~/.bashrc | grep "# CYGWIN LUA INSTALL")

if [[ -z $CHANGED ]] ; then
	echo "# CYGWIN LUA INSTALL" >> ~/.bashrc
	echo "export LUA_PATH=\"./?.lua;$LUA_LLIB_DIR/?.lua;$LUA_LLIB_DIR/?/init.lua;$LUA_LLIB_DIR/?.lua;$LUA_LLIB_DIR/?/init.lua\"" >> ~/.bashrc
	echo "export LUA_CPATH=\"./?.so;$LUA_CLIB_DIR/?.so;./?.dll;$LUA_CLIB_DIR/?.dll\"" >> ~/.bashrc
	echo "Lua path/cpath changed"
else
	echo "Previous changed detected, cancelled"	
fi

# do it now
export LUA_PATH="./?.lua;$LUA_LLIB_DIR/?.lua;$LUA_LLIB_DIR/?/init.lua;$LUA_LLIB_DIR/?.lua;$LUA_LLIB_DIR/?/init.lua"
export LUA_CPATH="./?.so;$LUA_CLIB_DIR/?.so;./?.dll;$LUA_CLIB_DIR/?.dll"

# test lua install
echo "############### Testing using require #################"
lua $START_DIR/test_require.lua

echo ""
echo "Enjoy Lua!"


```
  

**uninstall.sh**



```
#!/bin/sh

LUALOCAL=/usr/local

# dirs
LUAROCKS_CONFIG_DIR=$LUALOCAL/etc/luarocks
LUAROCKS_ROCK_DIR=$LUALOCAL/lib/luarocks
LUA_CLIB_DIR=$LUALOCAL/lib/lua
LUA_LLIB_DIR=$LUALOCAL/share/lua
rm -rf $LUAROCKS_CONFIG_DIR $LUAROCKS_ROCK_DIR $LUA_CLIB_DIR $LUA_LLIB_DIR

# bin files
LUAROCKS=$LUALOCAL/bin/luarocks
LUAROCKS_ADMIN=$LUALOCAL/bin/luarocks-admin
LUADOC=$LUALOCAL/bin/luadoc
LUAUNIT=$LUALOCAL/bin/lunit
rm -f $LUAROCKS $LUAROCKS_ADMIN $LUADOC $LUAUNIT

echo "Please manually remove LUA_PATH and LUA_CPATH in your .bashrc file"


```
  

**gd.patch**



```
--- Makefile	2006-05-04 09:03:48.000000000 +0800
+++ Makefile.update	2012-07-29 22:08:03.546875000 +0800
@@ -33,7 +33,7 @@
 # change the next ones.
 
 # Name of .pc file. "lua5.1" on Debian/Ubuntu
-LUAPKG=lua5.1
+LUAPKG=lua
 OUTFILE=gd.so
 CFLAGS=`gdlib-config --cflags` `pkg-config $(LUAPKG) --cflags` -O3 -Wall
 GDFEATURES=`gdlib-config --features |sed -e "s/GD_/-DGD_/g"`
@@ -67,11 +67,7 @@
 all: $(OUTFILE)
 
 $(OUTFILE): luagd.c
-	$(CC) -o $(OUTFILE) $(GDFEATURES) $(CFLAGS) $(LFLAGS) luagd.c
-	lua test_features.lua
-
-install: $(OUTFILE)
-	install -s $(OUTFILE) $(INSTALL_PATH)
+	$(CC) -o $(OUTFILE) $(GDFEATURES) $(CFLAGS) luagd.c $(LFLAGS)
 
 clean:
 	rm -f $(OUTFILE) *.o

```
  

**test\_require.lua**



```
#!/usr/bin/lua

local modules = {
	"zip",
	"luaXML",
	"lunit",
	"socket",
	"lfs",
	"logging",
	"luadoc",
	"md5",
	"date",
	"bit32",
	"lpeg",
	"luacurl",
	"luasql.sqlite3",
	"base64",
	"pack", 
	"profiler",
	"gd",
	"luacom",
}

for _,mod in ipairs(modules) do
	require(mod)
end

print "########## all passed #########"

```
  


#### 遇到的问题解释


**库路径**  




cygwin默认安装的Lua，库路径是 /usr/share/lua/5.1 和 /usr/lib/lua/5.1，luarocks编译时应该配置为 ./configure --prefix=/usr


但是在我的电脑上，如果这样做，luarocks在安装其他模块时会提示：


Error: Your user does not have write permissions in /  

-- you may want to run as a privileged user or use your local tree with --local.


于是我只好接受默认的/usr/local，这就带来了问题，即后面要修改 LUA\_PATH 和 LUA\_CPATH，另外cygwin中有些库是.so，有些是.dll，所以cpath比较特殊


**luagd Makefile**  




luagd的Makefile默认是编译不过去的。是因为debian和cygwin的差别造成的。大家看看gd.patch就知道哪里需要改了  




**卸载**


安装完之后要卸载就运行 ./uninstall 即可


**临时文件夹**


安装时使用了 ~/tmp/luainstall 作为临时文件夹，安装完后并没有删除，如果不想再看到它，请有空删除即可  




**运行**


我只能保证安装没有问题，具体是否能运行，还不知^\_^  




#### TODO


IUP或wxLua


Patch：整不明白GUI了，转战 MinGW，参见我的[下一篇文章](http://blog.csdn.net/windtailljj/article/details/7816182)  




  




  




