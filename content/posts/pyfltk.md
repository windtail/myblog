---
title: "Pyfltk"
date: 2025-05-25T10:57:43+08:00
draft: false
---

## Install

fltk是一个维护了很多年的C++跨平台GUI库，[fltk-rs](https://github.com/fltk-rs/fltk-rs) 是它的rust绑定，
可以做出很小的GUI程序，并且样子还可以 [fltk-theme](https://github.com/fltk-rs/fltk-theme)

那python有没有绑定呢，答案是肯定的，[pyfltk](https://pyfltk.sourceforge.io/install.php)，如果是Windows可以直接安装，

```shell
uv add pyfltk
```

linux就有一些复杂了，pyfltk最新版本（1.4.3.0）使用了fltk稳定版本1.4.3，操作系统中得安装fltk1.4.3，ubuntu 22.04中需要自己编译，
以下是安装过程。

### 安装fltk

```shell
git clone https://github.com/fltk/fltk.git
git switch branch-1.4
mkdir build
cd build
cmake -G Ninja -D FLTK_BUILD_SHARED_LIBS=ON -D CMAKE_POSITION_INDEPENDENT_CODE=ON -D CMAKE_BUILD_TYPE=Release ..
ninja
sudo ninja install
```

默认会安装到 /usr/local 中

### 安装pyfltk

从[这里](https://sourceforge.net/projects/pyfltk/files/latest/download)下载，然后解压，进入目录，执行

```shell
python setup.py swig
python setup.py bdist_wheel
```

这些会在根目录中生成 `dist/pyfltk-1.4.3.0-cp311-cp311-linux_x86_64.whl` 类似的文件，
最后就可以在对应Python版本的工程中使用 `uv pip install` 或 `pip install` 来安装这个wheel了。

## 文档和编程使用

建议参数fltk-rs的[文档](https://fltk-rs.github.io/fltk-book/)

因为fltk维护了很久，一般也只是用来写简单的GUI程序，这就非常适合将需求给ai，然后由ai自动生成代码即可！

## nuitka

使用nuitka编译后，运行后会报错，原因是nuitka会把 `__doc__` 属性设置为 `None`，导致fltk无法运行。原因代码在 `fltk.py` 这里

```python
sys.exit.__doc__ = \
r'''This is a sys.exit hooked by pyfltk. Discussion:

https://sourceforge.net/p/pyfltk/mailman/pyfltk-user/thread/87fsxgj3sh.fsf%40secretsauce.net/#msg37304779

 ''' + sys_exit_original.__doc__
```

把这个代码删除掉，编译后就可以正常运行了，我觉得 nuitka 应该把 `__doc__` 属性设置为空字符串，而不是 `None`
