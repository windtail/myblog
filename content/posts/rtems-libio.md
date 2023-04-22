---
title: "RTEMS LIBIO"
date: 2011-06-11T21:35:00+08:00
draft: false
---

 


为了让文件支持Unix标准的open, read, write, ioctl操作， RTEMS做了一层libio的封装，封装之后，用户不必了解libio过多，


只要会用就行了，我没有仔细地看代码，就是看了看console.c 和 相关的termio函数，然后做了几个实验得出，


若要使文件支持标准C的操作，只需要操作rtmes驱动函数中的最后一个参数，即 void \*arg，简述如下：


 


open、close函数： 将arg强制转换为 rtems\_libio\_open\_close\_args\_t \*，通过 mode和flag可获得打开时的标志，只读、只写、读写等


read、write函数： 将arg强制转换为 rtems\_libio\_rw\_arg\_t \*， 


     buffer为输出或输入缓冲区；


     count为输出缓冲区大小或输入数据个数；


     offset表示文件内偏移；


     bytes\_moved为返回的实际读出或写入的数据个数（由于设备原因，可能一次性不能读完或写完）；


  最后一次读操作bytes\_moved应为0，读写函数在没有错误的情况，都应返回RTEMS\_SUCCESSFUL


ioctl函数：将arg强制转换为rtems\_libio\_ioctl\_args\_t\*，


    command为命令；


    buffer为命令相关的参数，根据命令解释，也可作为命令的返回参数；


    ioctl\_return （暂时没搞清楚，貌似和ioctl函数返回值相同）


 


测试io的读写可以用RTEMS shell的 dd命令


测试读： dd if=/dev/ of=out.txt


             chmod 0777 out.txt


             cat out.txt


测试写： dd if=out.txt of=/dev/


 


注：读代码最快、最有效，以上的代码都在deviceio.c文件， 读一读发现特别地简单的！！以上叙述都不必看了……


