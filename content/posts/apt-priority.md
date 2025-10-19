---
title: "Apt Priority"
date: 2025-10-19T21:25:09+08:00
draft: false
tags: ["apt", "snap", "firefox", "thunderbird"]
---

## no snap!

在 `/etc/apt/preferences.d` 新建如下文件

### nosnap.pref

```text
Package: snapd
Pin: release a=*
Pin-Priority: -10
```

### firefox

```text
deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://mirrors.tuna.tsinghua.edu.cn/mozilla/apt mozilla main
```

```text
Package: firefox*
Pin: version 1:1snap1*
Pin-Priority: -1

Package: firefox*
Pin: origin mirrors.tuna.tsinghua.edu.cn
Pin-Priority: 1001
```

### thunderbird

```shell
sudo add-apt-repository ppa:mozillateam/ppa
```

```text
Package: thunderbird*
Pin: version 2:1snap1*
Pin-Priority: -1

Package: thunderbird*
Pin: release a=ppa:mozillateam/ppa
Pin-Priority: 1001
```

## 如何测试设置是否正确呢

设置完成后，需要执行 `sudo apt update`

然后再执行 `apt-cache policy firefox` 来测试版本的优先级
