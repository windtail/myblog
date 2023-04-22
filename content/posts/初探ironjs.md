---
title: "初探IronJS"
date: 2012-11-30T00:23:00+08:00
draft: false
---

话说最近有特别的需求，要用C#调用一种动态的语言执行一些经常改变的操作。由于我对Lua情有独钟，所以一开始就想到了它，了解了下LuaInterface，貌似问题挺多（[issue list](https://code.google.com/p/luainterface/issues/list)上有许多没有解决的defect），而且好像与时下很火的DLR没什么关系。


MS支持的Iron系语言看起来比较牛的样子，不过Python和Ruby我都没有接触过，最近也没有学门新语言的时间。搜了下IronLua，Google上有工程却没有代码。由于本人经常用JS写一些WSH的脚本（CMD太烂了），所以对JS还挺熟，于是IronJS就这样被我找到了！！


IronJS功能是非常地牛，但文档确实是少了点，除了[项目wiki](https://github.com/fholm/IronJS/wiki)和[blog](http://ironjs.wordpress.com/)上有一点，基本上就很难找到了，中文的更少了。刚试了下，感觉很好，记录下来，抛砖引玉，希望能引起大家的兴趣。


#### 准备工作


从github上下载编译好的dll或者把源代码下下来编译下，获得IronJS.dll，添加引用。


#### Hello IronJS




```
1         static void Main(string[] args)
2  {
3             var ctx = new IronJS.Hosting.CSharp.Context();
4             dynamic hello = ctx.Execute("(function (){ return {msg:'你好，IronJS'};})();");
5  Console.Out.WriteLine(hello.msg);
6  Console.Out.WriteLine(hello.noexist);
7             Console.ReadKey(true);
8         }
```


这段代码一看就懂，先是新建一个js运行的context，然后执行一js的函数，这个js的函数返回一个object，包含msg属性，利用.NET 4.0的dynamic关键字，我们可以很方便地引用成员，另外也可以看到当引用不存在的成员时会返回undefined，符合期望。


#### Access C# object




```
 1         class User
 2  {
 3             public string Name { get; set; }
 4  }
 5 
 6         static void Main(string[] args)
 7  {
 8             var ctx = new IronJS.Hosting.CSharp.Context();
 9             var user = new User() { Name = "张三" };
10             ctx.SetGlobal("user", user);
11  ctx.CreatePrintFunction();
12             ctx.Execute(@"
13  print(user.Name);
14  print(user.noexist);
15  print(typeof user.nodef == 'undefined' ? 'Yes' : 'No')");
16             Console.ReadKey(true);
17         }
```


第10行将user变量设置到js的全局变量中，第11行在js环境中创建了一个print函数，然后用js访问了user对象属性，并且在访问不存在的属性时，返回的undefined，完全符合期望。


#### JSON




```
 1 static void Main(string[] args)
 2  {
 3             var ctx = new IronJS.Hosting.CSharp.Context();
 4             var json = @"
 5  var configs = { 
 6  location:{x:0, y:0}, 
 7  size:{cx:100, cy:200},
 8  color:'red'
 9  };";
10 
11             var configs = ctx.Execute(json);
12 
13             Console.Out.WriteLine("x={0},y={1},cx={2},cy={3},color={4}",
14  configs.location.x,
15  configs.location.y,
16  configs.size.cx,
17  configs.size.cy,
18  configs.color);
19 
20             Console.ReadKey(true);
21         }
```


就是这么简单！如果用户程序需要一个配置文件，完全没有必要整XML，就算LINQ to XML简单，但是能简单到直接读上来执行一下吗。


#### Regular Expression


因为两年前据说IronJS不支持正则表达式，所以特意试下，现在真的是支持的！




```
1         static void Main(string[] args)
2  {
3             var ctx = new IronJS.Hosting.CSharp.Context();
4             ctx.Execute(@"var re = /(\d+)-(\d+)-(\d+)/;");
5             dynamic result = ctx.Execute(@"re.exec('2012-11-29');"); 
6             Console.Out.WriteLine("{0}年{1}月{2}日", result[1], result[2], result[3]);
7             Console.ReadKey(true);
8         }
```


睡觉了，上面试的几点基本能够满足我整一个配置文件，然后根据一个内嵌js语句的模板生成一个C文件的需求。有空再研究更多的功能吧。


