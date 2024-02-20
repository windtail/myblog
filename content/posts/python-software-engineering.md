---
title: "Python Software Engineering"
date: 2024-02-20T08:41:08+08:00
tags:
  - python
  - mypy
  - ruff
  - flit
  - pyproject
draft: false
---

## mypy

自从拜读了[Python is two languages now, and that's actually great](https://threeofwands.com/python-is-two-languages-now-and-thats-actually-great/)这篇文章之后，
我忽然就接受了Python是两门语言的这个事实。之前我一直很排斥给Python代码添加类型，“这么费劲，为啥我不用Rust或C++呢？” ，但是仔细想想，其实Rust和C++，甚至C都是两种语言:
  - Rust的macro和procedural macro
  - C++的template和concept
  - C的macro

究其原因，一般是因为写通用库时需要很多元编程的技巧，使得库足够通用，能够适应更多的情况，增加代码的复用性。
库代码一般都是功能比较确定的，并且写完后，很长时间都不会大改，就算代码复杂一点，也可以使用单元测试来保证正确性。

但应用的上层代码就不一样了，应用代码比较庞大，有很多人同时维护，并且功能经常由于需求的增加或改动，很容易在重构的时候出错。
静态类型有助于：
  - 增加代码的可读性
  - 编译器能够检查出更多的错误

mypy就是这样一个表达式类型检查器，据说连 Python 创始人都用这个。一般的类型提示都比较简单，Generic和Protocol略复杂一点：
  - Generic也就是泛型，Python 3.12 之后写法类似于C++/Rust的泛型
  - Protocol类似于接口，不要求继承自某个类型，只需要长像相同即可，和Golang的接口实现有点像

> 特别说明：mypy不喜欢相对import，根据我的理解，现在大家都推荐使用绝对import

代码根目录添加 `mypy.ini`

```ini
[mypy]
mypy_path = $MYPY_CONFIG_FILE_DIR
packages = your_package_name
check_untyped_defs = True

[mypy-your_package_name.ignore.import_.path.*]
ignore_errors = True
```

其中 `your_package_name` 是你的package名称，而 `your_package_name.ignore.import_.path.*` 是你想忽略的路径，
渐进式引入mypy时，一开始可以忽略很多，一步一步添加。但最后还可能会有一些工作生成的代码也需要被忽略。

### overload 和 pyi

如果一个函数有很多重载版本，即带有args和kwargs时，可以使用 overload 来标记可能的不同版本。

有的时候还可以把函数的类型声明放到一个同名的pyi文件中，例如 `a.py` 可以带一个同名的 `a.pyi`，
那么各种工具将会使用pyi文件来查找类型提示。

### py.typed

如果所有的代码都添加了类型提示，那么可以在package的目录中添加一个 `py.typed` 文件，来声明这个package自己已经有类型提示了，
如果不这样做，import时mypy会被报错。

## pyproject.toml 和 flit

setup.py或requirements.txt的形式官方已经标记为deprecated，但是官方并没有给一个推荐的方法。
对于绝大多数package来说，flit比较简单，推荐使用。

```toml
[build-system]
requires = ["flit_core >=3.2,<4"]
build-backend = "flit_core.buildapi"

[project]
name = "your_package_name"
authors = [
    { name = "your name", email = "your email" },
]
readme = "README.md"
dynamic = ["version", "description"]
requires-python = '>=3.8'
classifiers = ["Private :: Do Not Upload"]
dependencies = [
    "chardet==3.0.4"
]

[project.optional-dependencies]
test = [
    "pytest==7.4.0",
    "pytest-cov==4.1.0",
    "hypothesis==6.90.0"
]
dev = [
    "flit>=3.2,<4",
    "wheel",
    "mypy==1.8.0",
]

[project.urls]
Home = "your project url"

[project.scripts]
your_script_exec_name = "your_package_name.cmd:cli"
```

注意 `pyproject.toml` 中可以设置Python的版本，还可以设置dev和test需要的依赖，也可以声明console或gui的可执行脚本。

## ruff

最后对于一个大家一起维护的项目，相同格式化，相同的import顺序等有助于代码的阅读和合并。ruff运行速度很快，可以很好的解决这个问题。

`pyproject.toml`中的dev依赖可以添加ruff，然后ruff配置也可以写在 `pyproject.toml` 中，像这样

```toml
[tool.ruff]
line-length = 100
extend-exclude = ["generated", "other-exclude"]
extend-select = ["I"]

[tool.ruff.per-file-ignores]
"a.py" = ["E741"]
"__init__.py" = ["F401", "F403", "F405"]
```

## 总结

我一直以为一个大型项目，使用Python会导致很多运行时错误，因为编译时太多问题检查不出来了，
而实际项目中一般也没有足够时间撰写非常完整的单元测试，导致生产代码时不时地出现运行时错误。
我曾经想使用Golang或Rust替代Python，但是Golang，尤其是Rust，比Python要难一些，不容易招到合适的开发人员，
自从有了mypy/ruff，Python顿时又香了！