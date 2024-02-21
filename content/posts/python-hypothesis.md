---
title: "Python Hypothesis"
date: 2024-02-21T08:20:53+08:00
tags:
  - python
  - unittest
  - hypothesis
draft: false
---

当我们有了py.test这的单元测试框架后，fixture已经让写单元测试已经变得很容易了。
但是我们仍然要对多个输入进行各种组合，有没有一个库，可以通过指定每个输入的范围，自动产生多个输入的各种组合，
并自动测试边界条件呢？答案就是 [hypothesis](https://hypothesis.readthedocs.io/en/latest/) ，示例代码如下

```python
from hypothesis import given
from hypothesis import strategies as st

@given(
    region=st.sampled_from(list("XYKDRSGF")),
    offset=st.integers(min_value=0, max_value=19),
    bit=st.integers(min_value=0, max_value=7),
)
def test_bit(region, offset, bit):
    # use region, offset and bit as input to test
    ...

```

从以上代码可以看出，我们只是给了region，offset和bit的范围，并没有取各种边界值组合，
hypothesis自动处理了这些！！据说rust也有类似的库 [proptest](https://proptest-rs.github.io/proptest/proptest/index.html)，下次也试试。