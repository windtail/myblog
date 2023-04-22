---
title: "msys2 use self hosted mirror"
date: 2022-07-14T18:56:00+08:00
draft: false
---

## update mirrorlist.xxx

`C:\msys64\etc\pacman.d` 里的所有 mirrorlist.xxx 都改成只有自己的镜像地址（https）

## install pki

将自己的证书放到 `C:\msys64\etc\pki\ca-trust\source\anchors\xxx.crt`，然后执行

```shell
update-ca-trust
```

## update and install package

```
pacman -Sy             # 更新
pacman -Ss  pkg   # 查询包
pacman -S pkg      # 安装包
```

