---
title: "Try Pyo3"
date: 2024-10-27T18:57:29+08:00
draft: false
tags:
  - python
  - rust  
---

使用[pyo3](https://pyo3.rs)调用rust的函数，感觉比pybind11还要简单，
因为rust编译更简单，还有模板工程。

## 初始化

以下来自官方文档

```shell
# (replace string_sum with the desired package name)
mkdir string_sum
cd string_sum
python -m venv .env
source .env/bin/activate
pip install maturin
maturin init
maturin develop
```

经过以上命令，就可以直接在python中使用了

## 声明式创建模块和函数

```rust
use pyo3::prelude::*;

#[pymodule]
mod my_extension {
    use super::*;

    #[pyfunction] // This will be part of the module
    fn triple(x: usize) -> usize {
        x * 3
    }

    #[pyclass] // This will be part of the module
    struct Unit;

    #[pymodule]
    mod submodule {
        // This is a submodule
    }

    #[pymodule_init]
    fn init(m: &Bound<'_, PyModule>) -> PyResult<()> {
        // Arbitrary code to run at the module initialization
    }
}
```

这样写更简单的一些，省得还需要手动添加function和class

## 错误处理

返回 `PyResult<T>`，如果错误，则会在python那边抛出异常，
`pyo3::exceptions`中有内置的错误，比如`PyValueError::new_err("x is negative")`

`PyResult<T>`实际上就是`Result<T, PyErr>`，一些标准的错误已经可以直接转成PyErr了，
如果是自己的错误，实现from trait即可

函数的参数可以用signature来声明，例如以下方式，num默认是10，name是keyword only参数，
并且还支持args和kwargs，py_args在rust这边是 PyTuple，而 py_kwargs在rust这边是
Option<PyDict>

```rust
#[pyo3(signature = (num=10, *py_args, name="Hello", **py_kwargs))]
```

`class` 暂时没有用到，用到时再了解。
