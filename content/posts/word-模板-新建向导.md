---
title: "Word 模板 - 新建向导"
date: 2011-11-14T23:35:00+08:00
draft: false
---

向导
==


Word 2010 Bible 上说有的模板，在基于它新建文件时，可以弹出一个新建向导。然后，我就特别地想实现这个功能，比如新建若干个节，新建目录等。


跑到 http://word.mvps.org 上搜索了下，果然有方法，简单的说就是 在模板工程的ThisDocument中，新建一个Document\_New()过程，并实现之。复杂地说就是：


1. 双击 ThisDocument
2. 在右边打开的编辑区上方有两个组合框，选择左边那个框中的 Document，你会发现Word自动新建一个Document\_New过程
3. 添加自定义Form，做成向导即可


页面设置
====


向导做完以后，一般来说需要进行页面设置。我看了半天帮助文件，了解了Section.PageSetup的各成员，好不容易才搞定，后来发现只需要找个空文件，按页面设置要求，录制一个宏，然后拷贝一下即可。


**值得说明的是：**页面设置最后一页里，网格的每页的行数和跨度，在VBA中只能设置LinesPage，Word会根据这个值计算跨度=(PageHeight-TopMargin-BottomMargin)/LinesPage，用户可以在页面设置中修改跨度，并且Word也会作出响应（即一页可以不刚好是整数行），但是VBA做不到这一点，很让人崩溃~~


页眉和页脚
=====


页眉和页脚由Section.Headers和Section.Footers来设置，


1. wdHeaderFooterPrimary：奇数页
2. wdHeaderFooterEvenPages：偶数页
3. wdHeaderFooterFirstPage：首页


页脚中可以添加页码，页码用域 {Page} 来表示，要控制页码的样式或者重新编号，就需要用到


* Section.Footers.PageNumbers.RestartNumberingAtSection：是否在这节重新开始编号
* Section.Footers.PageNumbers.StartingNumber：编号开始
* Section.Footers.PageNumbers.NumberStyle：编号样式


**注意**：不需要Section.Footers.PageNumbers.Add 方法来添加页码，直接在 Footers(index).Range中添加Page域即可


**注意**：从第2节开始中需要设置页眉页脚的 LinkToPrevious 属性，并且在设置 Range属性时，需要先调用 Range.Delete，不然会有上一节的页眉页脚内容


添加目录与目录更新
=========


添加目录和更新目录比较特殊，直接插入和更新TOC域貌似不太好使。


**添加目录**：ActiveDocument.TablesOfContents.Add


**更新目录**：ActiveDocument.TablesOfContents(1).Update  




  

