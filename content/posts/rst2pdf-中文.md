---
title: "rst2pdf 中文"
date: 2018-01-12T13:49:00+08:00
draft: false
---

[上篇](http://www.cnblogs.com/windtail/p/8260193.html)说到用pandoc转换为reST为pdf是使用LaTeX作为中间格式的，而今天要说的rst2pdf貌似是直接转换为pdf的。


安装和调用
-----


rst2pdf目前只支持Python2.7，因此在创建virtualenv时应使用-p选项指定2.7的python，即




```
mkvirtualenv -p /path/to/python2.7 rst
workon rst
pip install rst2pdf
```


最简单的调用方式是




```
rst2pdf xxx.rst
```


然后默认就会生成xxx.pdf文件，当然中文这要是不好使的，因为默认样式是不支持中文字体的，生成的pdf里，中文都是黑块。


中文样式
----


rst2pdf有一些默认的样式，base、bodytext等等，我们还可以自己新增样式，也可以修改默认的一些样式的选项。对样式的选项修改类似于css，后面的样式选项可以覆盖前面的样式选项，所以我们一般不会去直接修改默认样式，而是在我们自己的样式文件中，覆盖默认的选项，在生成pdf的时候，可以一次性指定多个样式，后面的会覆盖前面的。


对于中文来说，我们就差一个字体的样式了，具体来说就是把下面的代码存储成 chinese.style




```
{
 "embeddedFonts": [
 [
 "simfang.ttf",
 "simhei.ttf",
 "simkai.ttf",
 "simsun.ttc"
 ]
 ],
 "fontsAlias": {
 "stdFont": "simfang",
 "stdBold": "simhei",
 "stdItalic": "simkai"
 }
}
```


注：有关字体的配置参见官方文档，这个配置对Linux和Windows应该都是好使的，前提是这些字体文件是真实存在的。Linux字体文件在 /usr/share/fonts 和 ~/.local/share/fonts 文件夹中。


此时，运行以下命令， 中文就可以显示出来了




```
rst2pdf -s chinese.style xxx.rst
```


 


配置文件
----


如果样式需要每次都指定，对于中文来说就有点不方便，rst2pdf支持一个配置文件，可以使用 --config 的方式传递配置文件路径，默认的配置文件路径是 ~/.rst2pdf/config


以下是官方方式给的配置文件示例+我改了stylesheets，stylesheet\_path，language




```
# This is an example config file. Modify and place in ~/.rst2pdf/config

[general]
# A comma-separated list of custom stylesheets. Example:
# stylesheets="fruity.json,a4paper.json,verasans.json"

stylesheets="chinese"

# Create a compressed PDF
# Use true/false (lower case) or 1/0
compressed=false

# A colon-separated list of folders to search for fonts. Example:
# font\_path="/usr/share/fonts:/usr/share/texmf-dist/fonts/"

font\_path=""

# A colon-separated list of folders to search for stylesheets. Example:
# stylesheet\_path="~/styles:/usr/share/styles"
stylesheet\_path="~/.rst2pdf/styles"

# Language to be used for hyphenation support

language="zh_CN"

# Default page header and footer
header=null
footer=null

# What to do if a literal block is too large. Can be
# shrink/truncate/overflow

fit\_mode="shrink"

# How to adjust the background image to the page.
# Can be: "scale" and "center"

fit\_background\_mode="center"

# What is the maximum level of heading that starts in a new page.
# 0 means no level starts in a new page.

break\_level=0

# How section breaks work. Can be "even", and sections start in an
# even page, "odd", and sections start in odd pages, or "any" and
# sections start in the next page, be it even or odd.

break\_side="any"

# Add a blank page at the beginning of the document

blank\_first\_page=false

# Treat the first page as even (default false, treat it as odd)

first\_page\_even=false

# Smart quotes.
# 0: Suppress all transformations. (Do nothing.)
# 1: Performs default SmartyPants transformations: quotes (including ‘‘backticks''
# -style), em-dashes, and ellipses. "--" (dash dash) is used to signify an em-dash;
# there is no support for en-dashes.
# 2:  Same as 1, except that it uses the old-school typewriter shorthand for
# dashes: "--" (dash dash) for en-dashes, "---" (dash dash dash) for em-dashes.
# 3: Same as 2, but inverts the shorthand for dashes: "--" (dash dash) for
# em-dashes,  and "---" (dash dash dash) for en-dashes.

smartquotes=0

# Footnote backlinks enabled or not (default: enabled)

footnote\_backlinks=true

# Show footnotes inline instead of at the end of the document

inline\_footnotes=false

# Cover page template.
# It will be searched in the document's folder, in ~/.rst2pdf/templates and
# in the templates subfolder of the package folder

# custom\_cover = cover.tmpl

# Use floating images.
# Makes the behaviour of images with the :align: attribute more like rst2html's

floating\_images = false

# Support the ..raw:: html directive
raw\_html = false
```


 使用以上的配置文件，我们需要将刚刚的 chinese.style 放到 ~/.rst2pdf/styles/ 文件夹中，以后再生成pdf的时候，就可以简单的使用和英文相同的方式了




```
rst2pdf xxx.rst
```


 默认设置下，rst2pdf的效果貌似要比pandoc的要好，以后不太要求的文档就用这个好了，长篇文档，估计还得使用sphinx


参考
--


[官方文档](http://rst2pdf.ralsina.me/handbook.html#configuration-file)：http://rst2pdf.ralsina.me/handbook.html


[某博客](http://blog.163.com/ar_cn/blog/static/14538308520104102716573/): http://blog.163.com/ar\_cn/blog/static/14538308520104102716573/


 


