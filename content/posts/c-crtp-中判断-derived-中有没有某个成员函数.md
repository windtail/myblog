---
title: "c++ CRTP 中判断 Derived 中有没有某个成员函数"
date: 2023-04-22T11:38:00+08:00
draft: false
---

```c++

// 省略 HasMember

template <Dervied>
class B {
  static_assert(HasMember<Derived>());
}

class A : public B<A> {
public:
  void Member();
}
```

这样的代码是编译不过的，因为A还没有完全定义时，static_assert就会fail，但是将static_assert放到某个函数里是可以编译过的。
