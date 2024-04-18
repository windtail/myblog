---
title: "Make exe like pip does"
date: 2024-04-18T19:37:09+08:00
draft: false
tags:
  - python
  - pip
---

当需要把python的脚本生成一个exe时，一般都是用pyinstaller或py2exe，但是我这个需求没有那个必要，
因为我是把python embeddable package直接给了用户，但是入口脚本我想是一个exe的形式，这样可以直接点击，
用户感觉可能会好一点。

所以我只是需要一个轻量级封装，pip安装后在Scripts文件夹中产生的exe就特别适合，[这里](https://stackoverflow.com/questions/35412392/how-can-i-use-setuptools-to-create-an-exe-launcher)有说明，
按照这个我写了段代码

```python
# coding: utf-8

import sys
from pathlib import Path

from pip._vendor.distlib.scripts import ScriptMaker

install_dir = Path.home() # anywhere you want

sm = ScriptMaker(
    None,  # source_dir - not needed when using an entry spec
    target_dir=install_dir.as_posix(),  # folder that your exe should end up inside
    add_launchers=True,  # create exe launchers, not just python scripts
)

# set the python executable manually
sm.executable = sys.executable.replace("pythonw", "python")

# create only the main variant (not the one with X.Y suffix)
sm.variants = [""]

# provide an entry specification string here, just like in pyproject.toml
# gui=True means lanuch by pythonw
sm.make("kuiide = module.file:func", {"gui": True})
```

> 注意：如果python.exe挪了位置，这种exe就不好使了，需要重新生成。因此这种方法，适合由nsis在安装时执行
