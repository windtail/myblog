---
title: "cmake使用ccache"
date: 2020-03-06T14:11:00+08:00
draft: false
---

来自 [stackoverflow](https://stackoverflow.com/questions/1815688/how-to-use-ccache-with-cmake)




```
find_program(CCACHE_PROGRAM ccache)
if (CCACHE_PROGRAM)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif ()
```


理论上distcc也可以用这种方式


