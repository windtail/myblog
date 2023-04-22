---
title: "IronJS与CSScript"
date: 2012-12-14T23:55:00+08:00
draft: false
---

[CSScript](http://www.csscript.net/)确实是个不错的东西，填补了WSH没有C#的空白，最重要的是大大扩展了脚本的功能，以前用JS写WSH的脚本，功能实在是弱了点，想把文件存成UTF-8，还不得不用ADOBD.Stream，着实不方便。


有了CSScript，以后写点编译用的简单脚本就方便了，用起来比Lua爽多了（当然，Lua本来就不是干这个的）。


CSScript本质上是将C#在线编译，然后执行，如果要执行一些运行时动态生成的代码，就没有这么方便了，毕竟C#不是动态语言，每一行代码都得编译成assembly才能运行。如果将CSScript和IronJS结合在一起，那所有问题就都解决了。




```
 1 using System;
 2 using Microsoft.CSharp;
 3 using IronJS;
 4 
 5 class Script
 6 {
 7     static void Main(string[] args)
 8  {
 9         var ctx = new IronJS.Hosting.CSharp.Context();
10         dynamic hello = ctx.Execute("(function (){ return {msg:'你好，IronJS'};})();");
11  Console.Out.WriteLine(hello.msg);
12  Console.Out.WriteLine(hello.noexist);
13         Console.ReadKey(true);
14  }
15 }
```


注意，必须添加 “Microsoft.CSharp”，必须将IronJS.dll放到csscript能找到的地方，否则会发生编译错误！有关代码的其他解释，参见[初探IronJS](http://www.cnblogs.com/windtail/archive/2012/11/30/2795588.html)


最后，感谢[Rain.](http://www.cnblogs.com/Lowlevel/)兄告诉我CSScript这个好东西


