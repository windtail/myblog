---
title: "std:sort segment fault"
date: 2020-06-30T18:52:00+08:00
draft: false
---

偶遇std::sort的segment fault的，不知如何下手，心中还在思考是不是编译器的bug，搜索了下才发现，compare函数不正确时，还真是可能segment fault，


具体就是 compare(a, b) 函数，a


