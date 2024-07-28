---
title: "Windows Gitlab Ci"
date: 2024-07-28T22:14:57+08:00
draft: false
---

## 命令行切换到 system 用户

Windows上默认gitlab-runner是以 system 用户运行的，如果要更改 ci 的运行配置，那就必须要以 system 用户来完成这些设置。
Linux是sudo即可完成，Windows上参考[这个](https://specopssoft.com/blog/how-to-become-the-local-system-account-with-psexec/)，
下载 [PsTools](https://download.sysinternals.com/files/PSTools.zip)，解压后，在`管理员`模式的cmd中运行以下命令

```shell
PsExec.exe -s -i pwsh.exe
```

## 更改 system 用户的目录

将以下注册表路径，更改为期望的目录，因为`System32`目录下，.net的应用将无法运行，例如 wixtoolset

```text
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList\S-1-5-18\ProfileImagePath
```

## 环境变量

如果要修改环境变量并保存呢？我只知道 `set` 可以临时改变环境变量，今天才知道还有一个 `setx`

设置一个环境变量：`setx /m name value`
添加值到一个环境变量：`setx -m name "%name%;value"`

> 注意 gitlab-runner 对 cmd 支持不好，可能会出错，应在配置中将 shell 改为 powershell或pwsh
> pwsh是更新的powershell，强列建议安装下这个，避免不必要的问题

## .gitlab-ci.yml的语法

参见 [这里](https://docs.gitlab.com/ee/ci/yaml/) 和 [这里](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)

## 其他可能的问题

- 路径过长
  - windows 10/11 上可以在组策略中将路径限制给去掉：计算机配置->管理模板->系统->文件系统->启用Win32长路径
  - git配置项：`git config --global core.longpaths true`
