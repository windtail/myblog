---
title: "Web Frontend Cache Control"
date: 2025-11-09T20:09:55+08:00
draft: false
tags: ["cache", "frontend", "nginx"]
---

前段时间，前端同事说我的nginx配置有误，导致发布前端后，不能及时更新，然后和他讨论过后，又和AI讨论了一会，
终于了解了缓存的机制，一直没有时间记录，今天忽然想起，记录一下，以免又忘记了。

## 前端开发和浏览器缓存

1. 前端有一些资源，编译时文件名称就带hash，只要hash不变，相关文件内容就不会变化。
   这种，nginx配置时，应告诉浏览器，永远缓存，`Cache-Control: public, max-age=31536000, immutable`
   浏览器请求了一次，便再也不会请求了，除非浏览器清了缓存。
2. 还有一些资源，文件名称不带hash，这种文件很少，比如`favicon.png`之类的，通常也不会变化，
   一般配置为`Cache-Control: public, max-age=3600`，即1小时。1小时之内，浏览器只会请求一次，
   1小时之后，会使用协商缓存，使用`last-modified-time`和`etag`判断是否返回304
3. 但是有一个特殊的文件，`index.html`，现代前后端分离开发中，这可能是前端唯一的入口文件，我们希望
   前端发布之后，尽快更新。所以这个文件一般被设置为`Cache-Control: no-cache`，
   每次访问时都要协商，使用`last-modified-time`和`etag`判断是否返回304
4. 最后一般就是服务器配置错误了，默认nginx也会有`last-modified-time`和`etag`，但是没有设置`Cache-Control`，
   浏览器仍然会尝试缓存以提高访问速度，这种有一个好听名字，叫启动式缓存，不同浏览器不一样，一般会根据`last-modified-time`计算
   出一个浏览器觉得合适的缓存时间，缓存时间内不会访问服务器，一般会导致更新不及时。这种配置应避免。

## 误区

我之前一直以为前端url，如果不使用hash的方式，所有请求都会过一下服务器，然后服务器最后fallback成index.html，
所以想给index.html设置较长的缓存。后来发现，这是错误的，前端自己改变url时，根本不会访问服务器，只是操纵浏览器的历史记录而已，
只有当用户按下刷新按钮时，才会访问服务器，此时若服务器没有对应的url，则会返回index.html。
