---
title: "Dll Path Mystery"
date: 2025-12-02T11:36:47+08:00
draft: false
tags: ["dll", "windows"]
---

## Dll Path Mystery

Windows dll的搜索路径和PATH一样，真是一个设计的败笔，当一些程序想把自己放到PATH，却又不小心把dll也放到PATH中了，
如果恰好和其他的exe需要的dll重名了，甚至是相似的dll，只是编译的flag不一样，特性略有差别，那就成谜了。

这个时候，相同的程序在一台电脑上好使，在另一台电脑上可能就不好使了，这个需要查询一下，到底加载了哪个dll，
[procexp](https://learn.microsoft.com/en-us/sysinternals/downloads/process-explorer) 这个来自微软的工具，
可以帮忙我们找到exe加载的所有dll。选择想看的exe，然后 `按下 Ctrl+D，或者点击 View > Lower Pane View > DLLs，以显示底部窗格。`
