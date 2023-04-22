---
title: "python简单处理xml文件"
date: 2018-04-23T18:57:00+08:00
draft: false
---

Python若是想从xml里读点信息，用BeautifulSoup可能会容易一点，但是如果要修改xml，BeatifulSoup就搞不定了，其实直接用lxml就好。




```
1 from lxml import etree
2 
3 tree = etree.parse("xxx.xml")
4 cfgs = tree.find('//component[@name="cmake-settings"]/configurations')
5 cfgs.clear()
6 cfgs.append(etree.Element("configuration", attrib={"CONFIG\_NAME":"Debug"}))
7 tree.write("xxx.xml")
```


 etree表示整个xml树结构，对其元素修改，就直接表现为对etree的修改，然后存储即可。一般的函数用法现查即可，只有XPath需要了解下。目前我只了解了以一下几项，基本够用：


* /：表示类似路径中的/，表示下一级
* //：表示后代，即下一级、下两级、等等等……
* [@=]：可以根据属性值来过滤
* \*：表示匹配所有，例如：//\*表示所有的后代


 


 


 
 


