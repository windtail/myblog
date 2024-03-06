---
title: "C++ std::filesystem::last_write_time"
date: 2024-03-06T09:19:09+08:00
draft: false
tags:
  - c++
  - filesystem
---

c++ `std::filesystem::last_write_time` 会返回文件的最后修改时，但是这个时间要怎么用呢，
不好意思，[cppref](https://en.cppreference.com/w/cpp/filesystem/file_time_type) 并没有给出。
有问题还得找 [stackoverflow](https://stackoverflow.com/questions/61030383/how-to-convert-stdfilesystemfile-time-type-to-time-t)，
也不知道c++委员会为什么总是能搞出一些不那么好用的东西，即使是c++20仍然是那么地不直白。

```cpp
const auto fileTime = std::filesystem::last_write_time(filePath);
const auto systemTime = std::chrono::clock_cast<std::chrono::system_clock>(fileTime);
const auto time = std::chrono::system_clock::to_time_t(systemTime);
```

忽然想到昨天看到的一个梗，“你所关注的问题，c++委员会将于c++45标准解决，请耐心等待……”
