---
title: "Mingw Boost Python"
date: 2023-12-30T21:32:47+08:00
draft: false
---

## install mingw boost-python

```
pacman -Su mingw64/mingw-w64-x86_64-python
pacman -Su mingw64/mingw-w64-x86_64-boost
```

## let cmake find it

cmake sometimes find wrong library and executable, 
add following lines to ensure libraries and executables in mingw

```cmake
SET(CMAKE_FIND_ROOT_PATH C:/msys64/mingw64 )

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

## add matching version suffix

simple line as bellow, will not work

```cmake
find_package(Boost COMPONENTS python REQUIRED)
```

```shell
cd C:/msys64/mingw64
find . | grep boost_python

# you may get following results
# ./bin/libboost_python310-mt.dll
# ./lib/libboost_python310-mt.a
# ./lib/libboost_python310-mt.dll.a

```

then, the following line will work

```cmake
find_package(Boost COMPONENTS python310 REQUIRED)
```
