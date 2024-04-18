---
title: "Qt Get Rid of Console Window"
date: 2024-04-18T19:47:33+08:00
draft: false
tags:
  - Qt
  - Windows
  - Mingw
---

Qt程序cmake编译出来，运行时会弹出控制台窗口，有人说这样

```cmake
add_executable(TargetName WIN32 main.cpp)
```

就不会有。但是好像没有什么作用。

[这个方法](https://stackoverflow.com/questions/2753761/how-do-i-tell-cmake-not-to-create-a-console-window)是可以的，

```cmake
target_link_options(TargetName PRIVATE -mwindows)
```
