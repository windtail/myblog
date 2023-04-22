---
title: "nginx如何修改charset"
date: 2018-05-25T15:35:00+08:00
draft: false
---

需求
--


将某个不符合json规范的响应 application/json;charset=gb2312 转成 application/json;charset=utf-8


ngx\_http\_charset\_module?
---------------------------


根据官方文档，ngx\_http\_charset\_module 只支持单字节的charsets，而gb2312是双字节的，显然不可以的。


iconv-nginx-module
------------------


根据 [iconv-nginx-module](https://github.com/calio/iconv-nginx-module) 的github说明和我的[上一篇博客](https://www.cnblogs.com/windtail/p/9088759.html)，应该可以编译出支持 iconv-nginx-module的nginx了，配置文件按如下设置




```
server {
	listen 9000;

    # disable cache completely
    expires off;

    location /api/v1 {
        proxy_pass http://192.168.7.215:8000;

        # only allow http 1.1 client
        proxy_http_version 1.1;

        iconv_filter from=gbk to=utf-8;
        iconv_buffer_size 8k;
        proxy_hide_header Content-Type;
        add_header Content-Type "application/json;charset=utf-8";
    }
}
```


 


