---
title: "C语言错误处理——setjmp & longjmp"
date: 2012-01-14T19:54:00+08:00
draft: false
---

  




C语言没有像Java那样的try catch处理异常错误的能力，不过可以用setjmp和longjmp两个函数实现错误处理的基本逻辑。  

setjmp(BUFFER)会将程序当前的寄存器状态保存到BUFFER数组里，这个数组用jmp\_buf定义：  





```
#include 
jmp\_buf BUFFER;
```
  

longjmp(BUFFER, n)将程序流跳到setjmp的位置，同时恢复BUFFER中保存的状态。第二个参数n为一个整数，当通过longjmp(BUFFER, n)跳转到setjmp位置时，setjmp函数的返回值为n；否则，如果是直接执行setjmp，则返回为零。根据这个特性，可以将整数n视为错误代码，这样在执行 setjmp(BUFFER) 会就可以知道是哪一种错误被触发了。  

一个小例子如下：  





```
#include 
#include 
jmp\_buf BUFFER;
 
void handle\_error()
{
 int err\_code = setjmp(BUFFER);
 if(err\_code != 0)
 {
 printf("Error code: %d\n", err\_code);
 }
}
 
void trigger\_error(int err\_code)
{
 longjmp(BUFFER, err\_code);
}
 
int main()
{
 handle\_error();
 trigger\_error(1);
 trigger\_error(2);
 
 return 0;
}
```
  

在上面的代码中，trigger\_error触发了两个错误，都被handle\_error捕获到了，这是个简单完整的错误处理的例子。由于保存运行状态的BUFFER数组要在不同的函数中使用，因此BUFFER要声明为全局变量，这是一个不太优雅的地方。  

  

  

转自：<http://programmingnote.com/blog/?p=179>  




