---
title: "Vcpkg Is Really Bad, Use Conan Instead!"
date: 2025-11-23T20:37:31+08:00
draft: false
tags:
    - vcpkg
    - conan
    - c++
---

## [vcpkg](https://github.com/microsoft/vcpkg)

微软出的C++包管理工具，看起来很好，也默认支持很多库，但是有很多概念和其他语言的包管理工具不同。

- 软件包的名称叫 `port` ！
- 安装很奇怪，需要先下载一个vcpkg的git仓库，然后再运行一个bootstrap脚本下载另一个可执行文件（vcpkg.exe），
  这两个要配合工具，它俩的版本要怎么维护呢？
- vcpkg的git仓库里包含了所有的portfile（也就是配方），按理说应该不需要再联网了，但是它每次configure都要联网
- 它没有lock文件，而是将vcpkg这个仓库的sha也作用户使用的一部分（搞了一个vcpkg-configuration.json），
  然后指定版本时默认只能指定>=,不能指定==，还需要再写一个overrides来指定版本
- 二进制包共享缓存没有提供统一的方式，windows，linux，跨平台共享不方便，而且默认只会写入一个缓存，
  不能同时指定本地和远程，因为根本没有远程的概念。但实际一般都想在本地缓存，这样会比较快，然后准备共享之后，再显式上传到远程仓库
- 二进制包的package-id，并没有明确的计算方式，Windows 10上编译的，相同的vs2022的编译器版本，到Windows 11上又要重新编译
- 将shared的编译方式和编译器搞到一起了，使用x64-windows-dynamic，然后所有库都是动态链接的，但实际上只有LGPL的需要动态链接，其他不需要
- 文档真的很差，并且还有的时候在维护，访问不了！

## [conan](https://conan.io)

conan虽然没有vcpkg的包多，但是自己写 recipe 很简单，因为文档非常地详细，所以配上cmake，要做的事情很少。
local cache和remote是分离的，本地试完了，再upload到服务器上即可。
更好的是服务器还有现成的 [Artifactory Community Edition for C/C++](https://docs.conan.io/2/tutorial/conan_repositories/setting_up_conan_remotes/artifactory/artifactory_ce_cpp.html#artifactory-ce-cpp)！

- 安装方便 `uv tool install conan`，并且明确说明了版本兼容的方式，即大版本一定兼容！
- package-id的生成方式很明确，并且还要可以在recipe中手动控制，主要是 os、settings和options
- 简单的可以使用 conanfile.txt，复杂的可以使用 conanfile.py，有了python之后，可以控制得很具体，各种if-else方便
- 每个依赖包，可以使用独立的options控制是否shared方式编译
- 如果包已经下载或编译过，本地cache中有，那么就不需要再联网了

更更好的是 conan 还提供了IDE的插件，比如 Clion, VSCode!

## 经验总结

如果微软出了一个工具，一定要谨慎使用，即使 vcpkg 的 star 远多于 conan！
如果它还声称是跨平台的，那就绝对不要用了，因为它一定是 Windows 优先的。
