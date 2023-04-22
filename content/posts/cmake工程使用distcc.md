---
title: "cmake工程使用distcc"
date: 2018-04-12T13:43:00+08:00
draft: false
---

distcc可以加速编译，但是遇到cmake可能就需要处理下。


问题
--


distcc在 /usr/lib/distcc 中放了各编译器的soft link（如cc/gcc等等），如果 /usr/lib/distcc 放到PATH最开始那么就会被先找到，不过我没有这样做，而是临时使用CC和CXX，如下



```
distcc-pump make -j$(distcc -j) CC="distcc cc"
```

但是对于cmake来说，cmake在configure的时候记录了编译器的绝对路径，编译命令是类似 /usr/bin/cc -o -c，所以distribute根本就不会发生


尝试1
---


既然是这样，那理所当然是应该把 /usr/lib/distcc 放到PATH最开始，这样 cmake就会记录 /usr/lib/distcc/cc 作为编译器，一切都很好，直到cmake尝试用这个编译器编译点代码（用于检测编译器的特性），编译就会报错（无法编译过去）。手动在这种环境就尝试编译，会提示没有使用distcc-pump，此时若使用 distcc-pump /usr/lib/distcc/cc 来手动编译是可以的。


尝试2
---


于是，我大胆的猜测下，把 /usr/lib/distcc 放到PATH最开始，并且 distcc-pump cmake ..，肯定就可以了，很不幸，这次cmake找到的居然是 /usr/bin/cc


通过 man distcc-pump，我发现可以使用 distcc-pump --startup来看看给后续命令的环境变量，它居然又把 /usr/bin加到了/usr/lib/distcc之前，再运行后续命令。我思考了下，问题应该是这样的， 当/usr/lib/distcc 放到PATH最开始时，cc被link到 distcc，当实际运行时，distcc并不知道cc在哪里，所以它需要把/usr/bin放到最开始，来找到真正的cc的位置，不管怎样，用 /usr/lib/distcc/cc 编译文件时， /usr/lib/distcc 是不能在PATH最开始的位置，否则编译出错，但我们又希望cmake找到 /usr/lib/distcc/cc


解决方案
----


经过两次尝试，需求就很明显了


1. /usr/lib/distcc 不能放在PATH最开始的位置，/usr/bin应放在开始位置，以便 /usr/lib/distcc/cc 能找到正确的cc
2. cmake 应找到 /usr/lib/distcc/cc，而不是 /usr/bin/cc


既然 /usr/lib/distcc 不能放在PATH最开始，又要让cmake使用 /usr/lib/distcc/cc，那只能是手动指定了，如下


 




```
cmake -DCMAKE_C_COMPILER=/usr/lib/distcc/cc -DCMAKE_CXX_COMPILER=/usr/lib/distcc/c++ ...
```


 


这样 /usr/lib/distcc/cc 在运行时， /usr/bin 在PATH的最开始，它也能正确调用真正在 /usr/bin/cc 去执行编译


 


