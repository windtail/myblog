---
title: "Python Current App Dir"
date: 2025-05-25T10:47:12+08:00
draft: false
tags: ["python"]
---

我们有的时候需要获取只前程序所在的目录，可能需要加载一外部的资源等等。如果只是脚本，可以使用 `__file__`，但很多时候我们会使用
`pyinstaller`或`nuitka`进行打包，使用以下函数可以完成获取应用所在目录的需求

```python
import sys
from pathlib import Path

def get_app_dir() -> Path:
    if "__compiled__" in globals() or getattr(sys, "frozen", False):
        # nuitka --standalone or pyinstaller
        return Path(sys.executable).parent
    else:
        return Path(__file__).parent
```

其中 `pyinstaller` 使用 `sys.frozen`，而 `nuitka` 使用 `__compiled__`
