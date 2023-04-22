---
title: "ConTeXt 标题前后的空白"
date: 2012-09-05T21:11:00+08:00
draft: false
---

由于标题字一般都挺大，所以默认时标题之间的空白比较大，尤其是当多个标题在一起的时，空白就显得格外地大！

 要去除空白可以这样做：\setuphead[chapter][before=**\nowhitespace**,after=**\nowhitespace**]

 当然，我们也可以手动地加一些空白，比如：\setuphead[chapter][before=\nowhitespace,after=\nowhitespace**\vskip5pt**]


