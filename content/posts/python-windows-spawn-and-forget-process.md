---
title: "Python Windows Spawn and Forget Process"
date: 2024-02-02T19:05:48+08:00
tags:
  - python
  - process
  - detach
draft: false
---

运行一个程序，然后忘记（Detach）它
[Taken from here](https://stackoverflow.com/questions/52449997/how-to-detach-python-child-process-on-windows-without-setsid)

```python
if 'nt' == os.name:
     p = subprocess.Popen(
         [sys.executable, '-c', cmd], 
         close_fds=True,  # close stdin/stdout/stderr on child
         creationflags=subprocess.DETACHED_PROCESS | subprocess.CREATE_NEW_PROCESS_GROUP | subprocess.CREATE_NO_WINDOW,
     )
```
