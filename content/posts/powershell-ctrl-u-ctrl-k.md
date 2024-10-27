---
title: "Powershell Ctrl+U Ctrl+K"
date: 2024-10-27T16:41:59+08:00
draft: false
tags:
  - powershell
  - pwsh
---

在 bash 中 ctrl-u/ctrl-k 是很常用的快捷键，但是 powershell 里没有，

```shell
notepad $PROFILE
```

在其中添加以下两行

```shell
Set-PSReadLineKeyHandler -Key Ctrl+u -Function BackwardDeleteLine
Set-PSReadLineKeyHandler -Key Ctrl+k -Function ForwardDeleteLine
```

参见[这里](https://stackoverflow.com/questions/75360258/how-to-clear-line-or-password-prompt-in-windows-powershell-like-ctrlu-in-linux)
和[这里](https://learn.microsoft.com/en-us/powershell/module/psreadline/about/about_psreadline_functions?view=powershell-7.4)
