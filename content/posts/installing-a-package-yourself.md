---
title: "Installing a package yourself"
date: 2011-08-01T22:18:00+08:00
draft: false
---

  





### Installing a package yourself



The following are the steps that you should follow to install a new LaTeX package into your own home directory.


1. Download the package file(s) from wherever they are available. Most packages are available from[CTAN](http://www.ctan.org/search/); enter appropriate keywords in the search fields to find the files.
2. Packages may be distributed in different ways. Many packages on CTAN, for instance, come with a.dtx file and a
.ins file. If the package you are installing comes with these files, you will have to process them with latex to create the actual files that make up the package. That is, type  

latex filename.dtx *and/or* latex filename.ins  

to unpackage the various .sty and other files in the package.
3. Create a directory texmf in your home directory, if there is not one there already.
4. Install the various package files into subdirectories of texmf as follows:
	* All .bst and .bib files into texmf/bibtex (or subdirectories)
	* All font-related files into texmf/fonts (or subdirectories)
	* All documentation files into texmf/docs
	* All other files (.sty, .cls, .tex, etc.) should go intotexmf/tex.


转自：http://hi.baidu.com/%C1%EE%BA%FC%D2%BB%B6%FE/blog/item/924e2732af35e3f51a4cffca.html  

  

