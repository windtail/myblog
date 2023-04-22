---
title: "ubuntu boost.python"
date: 2014-12-06T17:24:00+08:00
draft: false
---

安装boost（未尝试只安装 libboost-python-dev）




```
sudo apt-get install libboost-all-dev
```


新建hello\_ext.cpp，输入以下代码




```
 1 char const \*greet() {
 2 return "hello world";
 3 }
 4 
 5 #include 
 6 
 7 BOOST\_PYTHON\_MODULE(hello\_ext) {
 8 using namespace boost::python;
 9 def("greet", greet);
10 }
```


存储，使用以下命令行编译：




```
g++ -I/usr/include/python2.7 -c -fPIC hello_ext.cpp -o hello\_ext.o
g++ -shared -o hello_ext.so hello_ext.o -lpython2.7 -lboost_python
```


 


在hello\_ext所在目录，打开 python shell


>>> import hello\_ext


>>> print hello\_ext.greet()


 


注意事项：


- 要添加 -lpython2.7 和 -lboost\_python，否则会出现一个很复杂的函数找不到的问题，参见 http://stackoverflow.com/questions/1771063/no-such-file-or-directory-error-with-boost-python


很好的示例代码： https://github.com/TNG/boost-python-examples


