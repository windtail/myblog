---
title: "git push not configured with USE_CURL_MULTI"
date: 2011-03-15T21:32:00+08:00
draft: false
---

 


git 默认编译时，在push的时候会提示不能运行，因为编译的时候没有指定USE\_CURL\_MULTI，在哪里指定呢。


 


直接在Makefile的CFLAGS里加 -DUSE\_CURL\_MULTI，但是不好使！


最后上网搜到是在http.h里，果然最开始的时候加了一句 #undef USE\_CURL\_MULTI，难怪在命令行里加 -DUSE\_CURL\_MULTI，无效，把#undef改为#define，并在下面定义 #define DEFAULT\_MAX\_REQUESTS 5 即可（跟下面的curl版本大于多少多少一样）


 


最后，网上说之所以默认时会是这样，因curl早期版本有bug，会导致数据丢失，不过目前看来，我的这个还是好使的，我的系统是cenos 5.5，


curl-7.15.5-9.el5


