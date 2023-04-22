---
title: "cmake编译Qt5"
date: 2020-03-06T14:33:00+08:00
draft: false
---

官方文档在[这里](https://doc.qt.io/qt-5/cmake-manual.html)




```
cmake_minimum_required(VERSION 3.15)
project(XXX)

set(CMAKE_CXX_STANDARD 14)

find_package(Qt5 COMPONENTS Core Qml Quick Charts Widgets DBus REQUIRED)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# add_executable(XXX a.cpp a.h b.cpp ...)

target_link_libraries(XXX PRIVATE Qt5::Core Qt5::Qml Qt5::Quick Qt5::Charts Qt5::Widgets Qt5::DBus)
```


以上的脚本依赖一个环境变量，应把Qt5\_DIR设置为Qt5Config.cmake所在的目录，如果使用QtCreator打开CMakeLists.txt文件时，默认会传入一个QT\_QMAKE\_EXECUTABLE的变量，使用这个变量，我们就可以不用设置Qt5\_DIR了，但需要添加如下代码




```
if (DEFINED QT_QMAKE_EXECUTABLE)
    get_filename_component(_QT_USR_DIR ${QT_QMAKE_EXECUTABLE} DIRECTORY)
    get_filename_component(_QT_USR_DIR ${_QT_USR_DIR} DIRECTORY)
    set(Qt5_DIR ${_QT_USR_DIR}/lib/cmake/Qt5)
    set(_QT_USR_DIR)
endif()
```


如果项目使用了Qml，很可能会需要链接OpenGL库，但不知道为什么Qt5的cmake不会自动依赖这个库，需要添加如下代码




```
find_package(OpenGL COMPONENTS OpenGL REQUIRED)

target_link_libraries(XXX PRIVATE OpenGL::GL) # 需要安装 libgl1-mesa-dev库
```


 如果使用QtCreator，应把所有文件都加入到Target的Source中，否则QtCreator的工程中就看不到了，对于qml这种与编译无关的文件，可以使用如下方法，添加




```
file(GLOB_RECURSE QML_FILES "qml/*.qml")
target_sources(XXX PRIVATE ${QML_FILES})
```


 


