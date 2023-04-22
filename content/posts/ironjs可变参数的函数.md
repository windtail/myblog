---
title: "IronJS可变参数的函数"
date: 2013-02-24T22:23:00+08:00
draft: false
---

IronJS在github上[wiki](https://github.com/fholm/IronJS/wiki/Exposing-a-CLR-function-as-a-native-JavaScript-function)上有示例代码说明如何输出一个CLR的函数到JS环境中去，但是他只说明了怎样输出一个固定参数的函数，未提及可变参数的问题。甚至于IronJS.Native.Utils.CreateFunction()必须通过第2个参数将CLR函数的参数个数告知IronJS。


C#的Console.WriteLine()函数用起来还是比较爽的，如何在JS环境中创建一个print(format, …)函数呢？通过JS函数的arguments可以轻松搞定这件事情，代码如下：




```
using System;
using System.Collections.Generic;
using IronJS;
using IronJS.Runtime;

class Script
{
 static void print(BoxedValue arguments)
 {
 Func<string, object> arg = name =>
 {
 return arguments.Object.Members[name];
 };

 int length = (int)(double)arg("length"); // js中只有double
        if (length == 0)
 {
 return;
 }

 string format = arg("0").ToString();
 if (length == 1)
 {
 Console.WriteLine(format);
 return;
 }

 object[] objs = new object[length - 1];
 for (int i = 1; i < length; i++)
 {
 objs[i - 1] = arg(i.ToString());
 }
 Console.WriteLine(format, objs); 
 }

 [STAThread]
 static public void Main(string[] args)
 {
 var ctx = new IronJS.Hosting.CSharp.Context();

 ctx.SetGlobal("clrprint",
 IronJS.Native.Utils.CreateFunction>(ctx.Environment, 1, print));
 ctx.Execute("function print() { clrprint(arguments); }");

 ctx.Execute(@"print('Hello {0} {1}', 'world', 123);");
 Console.ReadKey(false);
 }
}
```


以上代码采用最新版的csscript测试通过，注意JS中只有double，没有int，在转int之前必须先转成double


