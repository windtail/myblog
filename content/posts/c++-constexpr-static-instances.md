---
title: "C++ Constexpr Static Instances"
date: 2024-07-20T21:32:58+08:00
draft: false
tags:
  - c++
---

下面这个代码是无法编译通过的

```c++
class A {
public:
    A(val) : val{val} {}
    
    static constexpr A a1{0};
    
private:
    int val;
};
```

不过稍加修改就可以了，参考[这里](https://stackoverflow.com/questions/24342455/nested-static-constexpr-of-incomplete-type-valid-c-or-not)

> `7.1.5p9` The `constexpr` specifier `[dcl.constexpr]` (n3337)

> >A `constexpr` specifier used in an object declaration declares the object as `const`. Such an object shall have literal type and shall be initialized.

```c++
class A {
public:
    A(val) : val{val} {}

    static const A a1;
    
private:
    int val;
};

constexpr A A::a1{0};
```
