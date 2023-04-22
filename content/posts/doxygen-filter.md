---
title: "doxygen filter"
date: 2012-05-12T12:25:00+08:00
draft: false
---

上一篇写到某位大哥用perl写了一个doxygen lua filter，（INPUT\_FILTER）自我感觉应该用lua来写。


昨天上网搜了搜，原来filter的原理还是很简单的，就是读源代码，然后向stdout输出转换过程序。不管什么语言都要转换成对应的C/C++的元素才能被doxygen理解。看看doxygen lua做了什么：


lua2dox example.lua > example.txt


（lua2dox是最后的filter主程序，doxygen调用它时会将程序文件名作为参数，lua2dox会将转换过的程序输出到stdout）


example.txt的内容如下：




```
/// @file
/// @brief a Doxygen::Lua example
    /// table is supported
    PARAMETER.a = 1;
    PARAMETER.b = 2; /// end of line comment is also supported
    PARAMETER.c = 3;
/// my name
AUTHOR = 'Alec Chen';
/// my email
EMAIL = 'alec@cpan.org';
/// @brief The factorial is an example of a recursive function
/// @param n a positive integer
/// @return return the product of all positive integers less than or equal to n
function factorial(n)
;
/// a function namespace
luaCharacter = inheritsFrom( nil, "luaCharacter" );
/// @brief luaCharacter:reset
function luaCharacter-reset()
;
/// @brief luaCharacter:destroy
function luaCharacter-destroy()
;
/// @brief luaCharacter:entry_point
function luaCharacter-entry_point()
;
```
  

是不是感觉很熟悉！ --! 被转换成了 /// 。我们完全可以写一个filter.lua，然后将Doxyfile里的 FILTER\_PATTERNS = \*.lua=filter.lua

并且一开始不了解doxygen的格式，甚至可以忽略文件参数，直接输出看看doxygen会得什么格式，比如将filter.lua置成如下




```
local str = [[///@file
///@brief Hello
///@details How are you

///@brief aaaa
///@param aa p1
///@param bb p2
function bbb(aa,bb);
]]

print(str)
```
  

这样的filter.lua将会导致任何的lua文件都能转换成相同的doxygen网页。

了解了原理之后，我们就可以自己动手写filter了，但是lua的语法其实并不简单的，完全映射到C/C++感觉还是有困难的，倒是跟Javascript有几分神似。所以用Lua来写这个filter很难做好。官网的FAQ里也写明了，对于一些与C/C++不太相似的语言，可以考虑修改doxygen源代码中的scanner.l文件，扫了一眼源代码，感觉可以参照源代码中的InsideJS做特殊处理的地方来使doyxgen支持lua，应该是具有可行性的。当然，也可以考虑像tcl一样，写一个专门的[Lexer](http://therowes.net/~greg/download/tcl-doxygen-filter/)来做filter（用flex或antlr），希望有时间，有能力的同志来完成这件事吧^\_^，暂时我还是用那个perl脚本比较好。  




