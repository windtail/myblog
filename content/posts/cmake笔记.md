---
title: "cmake笔记"
date: 2018-04-04T17:42:00+08:00
draft: false
---

 list 函数中的list变量要的是名称 generate expression只能在很少的地方用，if里不能用 equal比较数字，strequal比较字符串 字符串可理解为list，以;分隔，把3.0.6，整成list用string replace 注意各命令OUTPUT VARIABLE的位置，不太一致 没有返回值的概念，返回通过传入变量名称，然后set parent scope来完成，不想污染全局变量名字空间就要放function中 funtion没有return，所以if else层数会比较多，容易出错 cache变量都是全局的 findxxx会把变量值置为xxx-NOTFOUND STREQUAL比较 packagefindquietly未指定时是未定义，而不是false，if not会报错 传参数时，如果想把list作为一个参数传进去，必须加双引号，否则会把参数个数不匹配 中止用message fatal error 没有target link option的概念，但可以用target link libraries传入引号引起来的字符串，传入list会报错，pkgconfig的ldflags只能这么传进来 header only的库可以有target link option，但有link option的库就不能创建import 或interface库了 target TYPE property判断executable和library


