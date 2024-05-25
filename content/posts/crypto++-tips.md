---
title: "Crypto++ Tips"
date: 2024-05-25T17:44:39+08:00
draft: true
---

# 编译安装

说到加密，首先想到的当然是 openssl，但是当我想用mingw64的clang++编译时，发现还需要perl，下载perl发现还是不行，
得用msys的perl，也就是说要安装msys，要是能不要这个依赖就好了。

对于c++来说，还有另一个选择，[crypto++](https://github.com/weidai11/cryptopp)，编译使用makefile，
默认mingw64的g++可以编译通过，但clang++却不能，有一点bug，这个clang++也是被ms搞得有点复杂了。
makefile多少是有点难读懂的，并且最近也不用了，要是能用cmake就好了，幸好已经有其他人完成了这项工作
[cryptopp-cmake](https://github.com/abdes/cryptopp-cmake)，编译总算是顺利完成了。

# 使用入门

编译并安装后，cryptopp-cmake 提供了 cmake 的 config 文件，这用起来就简单了，大概是这样：

```cmake
list(APPEND CMAKE_PREFIX_PATH /path/to/cryptopp/install/dir)
find_package(cryptopp REQURIED)
target_link_libraries(you-target cryptopp::cryptopp)
```

虽然crypto++的[wiki](https://www.cryptopp.com/wiki/Main_Page)写得还算行，但我建议在编写c++代码前阅读 [这篇文章](https://petanode.com/posts/brief-introduction-to-cryptopp/#first-things-first-why-using-a-crypto-library-is-important)，
要点是：
- 各种加密、解密算法都是变换，source经过变化写到sink中
- source可以是
  1. 文件：FileSource
  2. 字符串：StringSource
  3. 二进制缓冲区：ArraySource
- sink和source相似，只是把`Source`换成`Sink`就好了
- 所有的api中，传入的指针都被take ownership，而引用不会

# 非对称加密

非对称加密，也就是常用的RSA算法，crypto++可以自己生成密钥对，然后存储到`FileSink`或`StringSink`中。
如果想用`openssl`生成的密钥会有一点复杂，不过幸好我也没有这个需求。
注意一点就好了，RSA加密后的密文与密钥的位数是相同的，例如`RSA 2048`加密后的密文就是2048位，也就是`2048/8`字节。

# AES加密

AES是一个常用的，可靠的加密算法。AES有很多种模式，需要了解下，建议看下[这篇文章](https://www.highgo.ca/2019/08/08/the-difference-in-five-modes-in-the-aes-encryption-algorithm/)，
要点就是，至少要用CBC模式，但是要正确地padding，最简单的选择也许是CTR模式。加密时需要
- 一个128/256位的密钥（KEY）
- 一个128位的随机数（IV）
