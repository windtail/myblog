---
title: "App Singleton"
date: 2025-05-25T10:32:05+08:00
draft: false
tags:
  - python
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

## Windows 重启问题

Linux上我们一般会把pid文件放到 /run/ 下面，这是一个内存映射的文件，当系统重启时，这个文件会被删除。
但是 Windows 上没有这种文件夹，当Windows 上重启时，并且程序在此时意外退出，这个pid文件在重启之后仍然存在，
如果恰好这个pid对应的应用程序是存在的，但是是别的应用程序，那么我们的目标应用就无法启动了。

最好在发现pid文件，并且pid对应的进程存在时，还要再检查一下这个进程是否是目标进程，如果不是，则删除pid文件，并启动目标进程。

## 更好的方案

上面的pid方案，在程序意外退出（或被杀）时，pid文件不会被删除，总是可能出现一些问题，导致无法启动。

Windows平台更好的方案是创建一个创建一个全局的Mutex，如果能创建成功，则说明程序没有运行，否则说明程序已经运行了。
Linux平台更好的方案是创建一个抽象的命名管道（\0开头），如果能创建成功，则说明程序没有运行，否则说明程序已经运行了。