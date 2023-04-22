---
title: "distcc加速内核编译"
date: 2018-04-04T13:35:00+08:00
draft: false
---

Linux内核编译实在是费时间的事，搞内核移植的时候总要编译，生命有一部分就浪费在等内核编译完成上，有心想买个HP的工作站，看了下Z840的价格，想想还是算了。distcc早就听说过，一直没有去试试，今天终于试了试，真是大赞啊！！下面说下如何配置，为了方便叙述，先定义几个称呼


* 我直接工作的电脑称为L
* 用来加速编译的电脑们称为A、B、C（它们也被称为compile farm）


安装
--




```
# L电脑
sudo apt install distcc distccmon-gnome distcc-pump

# A/B/C电脑
sudo apt install distcc
```


配置
--


在L/A/B/C电脑上更改 /etc/default/distcc内容如下（注释已经删除）




```
STARTDISTCC="true"

# 下面的 /16表示前面的IP地址16位有效，即192.168.xx.xx的IP都是接受的
ALLOWEDNETS="127.0.0.1 192.168.0.0/16"

LISTENER=""

NICE="10"

JOBS=""

ZEROCONF="true"
```


 然后执行




```
sudo systemctl restart distcc
```


  此时，若在任何一台电脑上执行 distcc --show-hosts，将会列出L/A/B/C电脑的IP或IPv6，格式为




```
:3632/32
:3632/16
:3632/8
:3632/32
```


 其中3632是端口，这个是默认端口，一般不用改变，'/'后面限制的任务数，默认按4倍来计算的，如果这个时候执行后面将会提到的distcc-pump就会报没有host具有,cpp属性（这可以认为是distcc的一个bug，不过人家在man distcc里说明了，zeroconf不支持lzo和cpp属性，github上也明确表示不会合并patch，所以zeroconf我觉得也就是可以用来debug或初始化）


在**L电脑**的打开 /etc/distcc/hosts，将刚才 distcc --show-hosts的输出写进去，并在每行后面加上,cpp,lzo，大约如下（**把+zeroconf注释掉**）




```
:3632/32,cpp,lzo
:3632/16,cpp,lzo
:3632/8,cpp,lzo
:3632/32,cpp,lzo
```


  使用hosts文件有几个好处（可能就是因为这些好处，以至于开发者都不去修正zeroconf的BUG）


* 各电脑按照配置高低排序，把配置高的放前面，当任务少时先分配给配置高的机器
* 各电脑的任务限制数可以手动更改（给A/B/C电脑一点剩余CPU）
* L电脑自己需要执行预处理任务，应该把自己放到靠后的位置，并适当限制任务数，甚至不要放到列表中去
* 手动增加,cpp,lzo属性，避开BUG


配置完成后，此时在L电脑上再次执行 distcc --show-hosts，输出就是/etc/distcc/hosts的文件内容


安装编译器
-----


所有电脑需要安装编译器，并且放到相同的位置（我认为不太必要，只要都在PATH环境变量中就可以了吧），最好都是相同的版本。


编译内核
----


编译内核时，一般会先export ARCH 和 CROSS\_COMPILE，这个不用改变，正常来即可，也可以放到命令行上，我比较喜欢export，最后编译执行命令是




```
distcc-pump make -j$(distcc -j) O=build-xxx CC="distcc ${CROSS\_COMPILE}gcc"
```


  这个时候可以打开 distccmon-gnome 看看壮观的景象，**几分钟**之后内核就编译好了！


后记
--


还有一个叫dmucs的东西，可以手动指定每台机器的power，然后自动执行负载均衡，并优先发给power强的机器，不过我配置的半天都不好使，而且我觉得配置起来也比较麻烦，就放弃了。有兴趣的同志可以研究下。


补记
--


在编译Linux内核某版本时，总是报远程无法编译过，但本地可以编译过，然后后面就会使用plain distcc mode，也就是本地预处理，远程编译的方式。这种方式显著降低编译速度，因为大量工作由本机来完成，而我本机的性能还不如编译服务器，一开始我也以为配置的问题，后来 man include\_server告诉我，可以 export DISTCC\_FALLBACK=0，然后远程编译的错误信息就会打出来，便于诊断，发现是下面的代码的问题




```
#define __gcc_header(x) #x
#define _gcc_header(x) __gcc_header(linux/compiler-gcc##x.h)
#define gcc_header(x) _gcc_header(x)
#include gcc\_header(\_\_GNUC\_\_)
```


 


也就是说distcc预处理时无法知道要发送哪个头文件，并且没有检测出这个问题，导致远程编译时报 linux/compiler-gcc##x.h 未找到，这个问题work around的方法是，根据打印消息，把源代码临时改掉，不要因为这一个问题，导致编译速度显著下降。


另外 CC=distcc xxxx，如果xxxx不是绝对路径，那在远程编译服务器上也会按PATH去搜索，不过要注意PATH在远程是什么，如果是绝对路径就不需要搜索了，直接也使用绝对路径。


