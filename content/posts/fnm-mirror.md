---
title: "Fnm Mirror"
date: 2024-09-01T10:41:48+08:00
draft: false
---

```shell
fnm env --use-on-cd | Out-String | Invoke-Expression
fnm completions --shell power-shell | Out-String | Invoke-Expression
$env:FNM_NODE_DIST_MIRROR="https://npmmirror.com/dist"
```
