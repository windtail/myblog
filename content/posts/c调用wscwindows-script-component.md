---
title: "C#调用WSC（Windows Script Component）"
date: 2012-11-26T23:57:00+08:00
draft: false
---

WSC是一个很老的东西了，我现在是想用C#去执行一个WSH（javascript）的脚本，却又不想通过Process却调用cscript，直接call函数最好了。


![](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)WSC


```
 1 </spanxml version="1.0" encoding="utf-8"?>
 2 <component>
 3 </spancomponent error="true" debug="true"?>
 4 <registration 
 5 description="test"
 6  progid="test.wsc"
 7  clsid="{F0590009-B8C7-4B69-9DA9-6E1919F07936}" 
 8 />
 9 <public>
10 <method name="hello"> 
11 method>
12 public>
13 <script language="javascript">
14 </span>
<span style="color: #008080;">15</span> <span style="color: #808080;"> function hello() {
</span><span style="color: #008080;">16</span> <span style="color: #808080;"> return "Hello WSC";
</span><span style="color: #008080;">17</span> <span style="color: #808080;"> }
</span><span style="color: #008080;">18</span> <span style="color: #0000ff;">
19 script>
20 component>
```



将上面的代码保存为 test.wsc，然后右键选择注册。接着就可以去C#中调用了，这里用了.Net 4.0的 dynamic类型




```
Type TestType = Type.GetTypeFromProgID("test.wsc");
dynamic test= Activator.CreateInstance(TestType );
Console.Out.WriteLine(test.hello());
```


不过，如果你想传参数的话，问题就变得复杂了。C#的class是managed，javascript是访问不到，不过传个值还是可以的，比如int的值


暂时没时间研究在managed和unmanaged之间转换问题，哪位仁兄会的话，不妨分享下^\_^


#### x64问题


在x64位的Win7上右键注册wsc是不好使的，因为默认是注册system32下面的scrobj.dll，64程序访问就会未注册的问题。


据说用下面的语句注册和卸载就可以了（有待实验）




```
"C:\WINDOWS\SYSWOW64\REGSVR32.EXE" /i:"%1" "C:\WINDOWS\SYSWOW64\scrobj.dll"
```




```
"C:\WINDOWS\SYSWOW64\REGSVR32.EXE" /u /n /i:"%1" "C:\WINDOWS\SYSWOW64\scrobj.dll"
```


#### 调用js的其他方法


* IronJS：使用DLR，以类似IronPython和IronRuby的方式运行，据介绍速度还挺快
* rhino：codeproject上有[一篇文章](http://www.codeproject.com/Articles/41792/Embedding-JavaScript-into-C-with-Rhino-and-IKVM)介绍如何在C#中用rhino去执行javascript

