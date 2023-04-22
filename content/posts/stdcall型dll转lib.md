---
title: "__stdcall型dll转lib"
date: 2012-08-17T21:51:00+08:00
draft: false
---

    关于dll转lib文件的方法，网上的文章很多，但是我这次转的dll，输出函数是以 \_\_stdcall 打头声明的。稍有不一样，顺便将网上的文章总结下。

 #### 转换环境

     VS2008

 #### 转换步骤

 * 打开 Visual Studio 2008 命令提示
* 将\VC\bin 和 \Common7\IDE 加入到PATH路径：set PATH=%PATH%;\VC\bin;\Common7\IDE;
* dumpbin /exports xxx.dll>xxx.def
* 将xxx.def文件中以“ordinal hint RVA name”为表头的表中的 name 和 original 提取出来
* 将xxx.def文件修改为如下：

 
>  LIBRARY "xxx"
> 
>  EXPORTS
> 
>  **@=** @
> 
>  

 * lib /def:xxx.def 可生成 xxx.lib文件

 #### 解释

 \_\_stdcall的函数，在声明extern "C" 时，导出的函数名类似这样：\_func@16 

 * 前面加 "\_" （注意xxx.def文件中，这个下划线不要加，导出时会自动加上的）
* func：函数名
* 中间加 "@"
* 16：函数的所有参数总共占用的字节数

 注：不是\_\_stdcall声明的dll，将上述文字中的红色部分去掉即可。


