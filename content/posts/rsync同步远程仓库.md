---
title: "rsync同步远程仓库"
date: 2018-02-01T18:06:00+08:00
draft: false
---



```
rsync -aqzH --delete --delay-updates   rsync://elpa.emacs-china.org/elpa/ elpa
```


同步elpa到本地的elpa目录


