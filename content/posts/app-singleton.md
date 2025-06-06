---
title: "App Singleton"
date: 2025-05-25T10:32:05+08:00
draft: false
---

有时候，我们希望应用程序只运行一个，即单例。

## 方案

我们可以这样做，每次启动程序就生成一个文件，并把当前的pid存储到这个文件中。如果启动时发现这个文件存在，则说明已经有一个程序在运行了，直接退出。
当然这样有点想简单了，还需要改进一下：

- 读写文件时需要加锁，避免同时写入的情况
- 检查文件存在后，还需要读一下文件内容，如果pid不存在，则说明程序已经意外退出了，应写入新的pid并正常启动程序

当我们使用Python编程时，有没有一个包已经实现了这个功能呢？答案是肯定的，[pid](https://github.com/trbs/pid/)，并且它是跨平台的！

## pid

```shell
uv add pid
```

```python
from pid import PidFile
import os

with PidFile('foo') as p:
  print(p.pidname) # -> 'foo'
  print(p.piddir) # -> '/var/run' But you can modify it when initialize PidFile.
  print(os.listdir('/var/run')) # -> ['foo.pid']

# pid file will delete after 'with' literal.
```

```python
from pid.decorator import pidfile

@pidfile()
def main():
  pass

if __name__ == "__main__":
  main()
```
