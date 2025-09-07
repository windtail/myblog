---
title: "Upgrade Ubuntu"
date: 2025-09-07T13:48:41+08:00
draft: false
tags:
  - linux
  - ubuntu
  - upgrade
---

## 升级ubuntu

因 Tauri 2.0 需要至少 22.04 LTS，所以没有办法必须要升级了。重装当然更麻烦，但是内网要升级，需要访问哪些网站，如果中间失败会不会就挂了，不要snap怎么做。
这些 AI 都告诉我了，这里总结一下。

### 禁止snap

我不太喜欢snap，所以我当前没有安装任何snap包。如果安装了需要先删除，然后再把 snapd 也删除掉。最后要加一个配置，避免Ubuntu自动给加上snap。

`/etc/apt/preferences.d/nosnap.pref`

```text
Package: snapd
Pin: release a=*
Pin-Priority: -10
```

### 内网需要访问哪些网站

- changelogs.ubuntu.com
- motd.ubuntu.com
- mirrors.tuna.tsinghua.edu.cn

### 内网代理的设置

在命令行中设置 http_proxy, https_proxy, no_proxy，后面再调用 sudo 时，需要加上 `-E` 选项，例如 `sudo -E do-release-upgrade`

### 升级现在系统的包到最新版本

```shell
sudo apt update
sudo apt full-upgrade -y
sudo apt --fix-broken install
sudo apt autoremove --purge -y
sudo apt clean
```

> 如果有些包，你不想要了，一定要提前删除，否则升级出问题了后面会难办一点！


### 准备升级

```shell
sudo apt install -y update-manager-core
sudo do-release-upgrade -c
```

这一步应该会提示你有可用升级。

### 升级

```shell
sudo do-release-upgrade
```

如何运气好，那么一下就会升级成功了。如果运气不好，参见后面写的错误处理。


## 升级中遇到问题

因为我之前安装了context（TeX的一种），然后升级的时候就一直卡在 `mtxrun --generate` 不动了。很可能是需要联网下载，然后又下载不下来。
这个时候，只能 `Ctrl+C` 停止了，并且还需要把 dpkg 的进程 kill 掉。此时如果想卸载这个context，也是不允许的，会提示需要先执行
`dpkg --configure -a` 来完成升级，但是如果真的执行这个命令，又会卡住在 `mtxrun --generate`

这个时候可以到 `/var/lib/dpkg/info` 中，使用 `ag mtxrun` 找到在哪里会执行 mtxrun，在 `context.postinst` 中，于是我们把
`mv context.postinst context.postinst.bak`，此时再执行 `dpkg --configure -a` 就可以正常完成了。

升级结束了，我赶紧把 context 卸载了，但是我又不知道谁依赖了它。先安装 aptitude，然后执行 `aptitude why context`，它就告诉我
是 pandoc 依赖了它，我赶紧把 pandoc 卸载了，如果还需要重新安装就可以了。