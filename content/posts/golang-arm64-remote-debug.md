---
title: "golang arm64 remote debug"
date: 2021-12-30T17:14:00+08:00
draft: false
---

golang交叉编译非常容易，但是远程debug却不是那么容易，有人说要用gdb来debug，没有ide支持，体验不是很好。如果有幸是在arm64上运行程序，那么可以用[delve](https://github.com/derekparker/delve)（delve目前不支持arm32）


编译delve
-------




```
git clone https://github.com/derekparker/delve
cd delve
GOOS=linux GOARCH=arm64 go build -mod=vendor -o dlv cmd/dlv/main.go
```


然后把生成的dlv拷贝到板上的 /usr/bin/ 或其他位置


GoLand配置
--------


Edit Configurations... 点左侧+号，添加 Go Remote


其中Host改为板子的IP （注意下面的帮助)


编译应用程序
------




```
GOOS=linux GOARCH=arm64 go build gcflags "all=-N -l" xxx
```


板上启动
----




```
dlv --listen=:2345 --headless=true --api-version=2 --accept-multiclient exec ./xxx
```


 


最后在GoLand中点刚才添加的调试配置的启动按钮就可以了！


 


