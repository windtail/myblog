---
title: "Qt5 CMake cross compile"
date: 2018-11-30T17:21:00+08:00
draft: false
---

![](http://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif)![](http://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)


```
cmake_minimum_required(VERSION 2.8)

if (${ARM})
 set(CMAKE\_SYSTEM\_NAME Linux)
 set(CMAKE\_SYSTEM\_PROCESSOR arm)

 set(CMAKE_STAGING_PREFIX $ENV{HOME}/dev/kndos/rootfs)
 set(CMAKE\_SYSROOT ${CMAKE\_STAGING\_PREFIX})
 set(CMAKE_FIND_ROOT_PATH /usr/lib/arm-linux-gnueabihf ${CMAKE\_STAGING\_PREFIX})
 set(CMAKE_LIBRARY_ARCHITECTURE arm-linux-gnueabihf)

 set(CMAKE_C_COMPILER arm-linux-gnueabihf-gcc)
 set(CMAKE_CXX_COMPILER arm-linux-gnueabihf-g++)

 set(CMAKE\_FIND\_ROOT\_PATH\_MODE\_INCLUDE ONLY)
 set(CMAKE\_FIND\_ROOT\_PATH\_MODE\_PACKAGE ONLY)
 set(CMAKE\_FIND\_ROOT\_PATH\_MODE\_LIBRARY ONLY)
 set(CMAKE\_FIND\_ROOT\_PATH\_MODE\_PROGRAM NEVER)
endif ()

project(tqml)

set(CMAKE_CXX_STANDARD 14)

set(CMAKE\_AUTOMOC ON)
set(CMAKE\_AUTORCC ON)
set(CMAKE\_AUTOUIC ON)

find\_package(Qt5 COMPONENTS Widgets Qml Quick REQUIRED)

if (${CMAKE\_CROSSCOMPILING})
 set(HOST_QT_BIN /usr/lib/x86_64-linux-gnu/qt5/bin)
 set\_target\_properties(Qt5::uic PROPERTIES IMPORTED\_LOCATION ${HOST\_QT\_BIN}/uic)
 set\_target\_properties(Qt5::moc PROPERTIES IMPORTED\_LOCATION ${HOST\_QT\_BIN}/moc)
 set\_target\_properties(Qt5::rcc PROPERTIES IMPORTED\_LOCATION ${HOST\_QT\_BIN}/rcc)
endif ()

add\_executable(${PROJECT\_NAME} main.cpp main.qml main.qrc)
target\_link\_libraries(${PROJECT\_NAME}
 Qt5::Widgets
 Qt5::Qml
 Qt5::Quick)
```


CMakeLists.txt
其实只需要注意一个问题即可，其他的都有上位机相同，CMAKE\_FIND\_ROOT\_PATH\_MODE\_PROGRAM虽然设置成了NEVER，但是并不会改变Qt5::uic/moc/rcc的查找方式，需要覆盖下，使用上位机的uic/moc/rcc


注：我的rootfs就是使用debian-bootstrap构造的rootfs


 


