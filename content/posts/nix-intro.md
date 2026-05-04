---
title: "解密 Nix：从函数式语法到系统构建的核心逻辑"
date: 2026-05-04T18:22:42+08:00
draft: false
tags:
  - nix
---

对于初学者来说，Nix 往往像是一团迷雾。它的语法看似简单却又古怪，它的概念（Derivation, Package, Module...）似乎在指代同一件事，却又各司其职。

要真正掌握 Nix，我们需要“降维打击”：**先看它的语法本质，再看它如何用这些语法构建出整个世界。**

---

## 第一部分：Nix 语言——为配置而生的原子

Nix 是一门**纯函数式、惰性求值**的配置语言。在 Nix 的世界里，一切皆表达式，每一段代码都会返回一个值。

### 1. 基础数据类型
Nix 的类型系统非常精简，完全服务于文件处理和配置描述：

*   **数值 (Numeric)**: `1` (Integer) 或 `3.14` (Float)。
*   **布尔 (Boolean)**: `true` / `false`。
*   **路径 (Path)**: Nix 的灵魂。`./src/main.c` 是相对路径。
    *   路径不是字符串，没有引号 
    *   相对路径是相对nix文件所在目录，相对路径在concat之前会先展开成绝对路径
    *   **注意：** 当路径被插值到字符串中（如 `"${./file}"`），Nix 会自动将该文件拷贝到 `/nix/store` 并返回其不可变的哈希路径。
*   **字符串 (String)**: 
    *   单行用双引号 `"`。
    *   多行用一对双单引号 `''...''`，它会自动处理缩进，是写内联 Shell 脚本的神器。
    *   字符串可以使用 `+` 连接到一起
*   **列表 (List)**: 使用**空格**分隔，如 `[ 1 "hello" ./file ]`。使用 `++` 连接到一起

### 2. 属性集 (Attribute Set)
这是 Nix 最核心的结构，本质是键值对（Key-Value）。
*   **基础**: `{ a = 1; b = "text"; }`。
*   **递归 (`rec`)**: 允许集合内部属性互相引用。`rec { name = "hello"; full = "${name}-v1"; }`。

### 3. 函数魔法与解构
Nix 的函数非常强大，函数只接受一个参数，两个参数的函数，是通过第一个函数返回一个函数来完成(Currying)，如 `x: y: x + y`。
但一般传复杂参数都是通过arguemtn set来传：
*   **参数集解构 (Argument Set)**: 函数可以直接解构传入的属性集。
    *   `{ x, y }: x + y` 接收一个包含 `x` 和 `y` 的集合。
    *   `{ x, y ? 2}: x + y` 允许你设置一个默认值。
*   **省略号 (`...`)**: `{ x, ... }` 表示除了 `x` 之外，还可以接受其他我不关心的属性，防止因参数冗余而报错。
*   **@ 语法 (Aliasing)**: 这是高级玩家常用的技巧，它允许你既解构参数，又保留对整个原始集合的引用。
    ```nix
    # input 代表整个集合，同时解构出 version
    myFunc = input@{ version, ... }: {
      v = version;
      all = input; # 依然可以访问整个原始对象
    };
    
    # 也可以放到后面
    myFunc = { version, ... }@input: {
      v = version;
      all = input; # 依然可以访问整个原始对象
    };    
    ```

### 4. 变量与逻辑控制

* Let 绑定 (Let-in)：用于定义局部变量，作用域仅限于 in 之后的表达式。
* Inherit 关键字：一种简写语法，用于将当前作用域的同名变量快速拉入属性集中。
* With 语句：将一个属性集的所有属性引入当前作用域，常用于简化访问 pkgs 下的内容，但可能导致命名冲突。
* 条件表达式：使用标准的 if ... then ... else ... 结构，且 else 分支是强制要求的。

---

## 第二部分：四大核心概念的深度拆解

当我们理解了语法，再看 Nix 的核心概念，就会发现它们只是**具有特定结构的 Attribute Set**。

| 概念 | 语言层面（是什么） | 功能层面（干什么） |
| :--- | :--- | :--- |
| **Derivation** | 一个由 `derivation` 函数生成的 Set，包含 `name`, `builder`, `system` 等固定属性。 | **“构建说明书”**。描述了如何从源码变出软件。它会被实例化为 `/nix/store/*.drv` 文件。 |
| **Package** | 一个特定的 Derivation，其结果包含 `outPath`（指向安装路径）。 | **“软件产物”**。你最终运行的二进制文件或调用的库，存在于 `/nix/store/哈希-名称` 下。 |
| **App** | 一个遵循 Flake 规范的简单 Set，包含 `type = "app"` 和 `program` 路径。 | **“执行入口”**。定义了如何“跑”这个软件。一个 Package 包含 10 个工具，App 决定哪个是默认启动项。 |
| **Module** | 一个包含 `options`, `config` 和 `imports` 的函数或 Set。 | **“配置逻辑”**。负责系统级的声明（如：开启 Nginx 服务、配置防火墙）。它负责将各种 Package 组装成一个运行中的系统。 |

---

## 第三部分：横向联动——它们是如何工作的？

让我们把这些点串联起来，看看一个完整的流程：

1.  **编写表达式**: 你写下一个 Nix 函数，接收一个 **Argument Set**（通常包含 `pkgs`, `lib` 等）。
2.  **定义 Derivation**: 函数内部利用 `stdenv.mkDerivation` 产生一个 **Attribute Set**，指定 `buildInputs` 和 `src`。
3.  **计算路径**: Nix 扫描这个 Set，根据内容生成唯一的哈希，确定 **Package** 的未来居所（`outPath`）。
4.  **构建系统**: 如果你是在 NixOS 里，**Module** 会接过这个 Package，把它写进系统路径，或者通过 **App** 定义让你可以直接 `nix run` 启动它。

### Derivation

在 Nix 中，尤其是使用最常用的 `stdenv.mkDerivation`（它是对底层 `builtins.derivation` 的封装）时，有一系列约定俗成的属性。

理解这些属性，就能读懂 90% 以上的 Nix 软件包定义。我们可以将它们分为 **依赖管理、构建生命周期、环境控制** 三类：

### 1. 依赖管理属性 (Dependencies)
这些属性决定了构建环境里有哪些“工具”和“库”。

| 属性 | 用途 | 举例 |
| :--- | :--- | :--- |
| **`nativeBuildInputs`** | **编译时需要的工具**。跑在开发机架构上的程序（如 `cmake`, `pkg-config`, `gcc`）。 | `[ cmake ninja ]` |
| **`buildInputs`** | **运行时需要的库**。目标架构的库文件，会被链接到产物中。 | `[ glib qt6.qtbase ]` |
| **`propagatedBuildInputs`** | **传递性依赖**。不仅自己编译要用，谁依赖我，谁也得自动带上这些依赖（常用于 Python 包或特定库）。 | `[ numpy ]` |
| **`checkInputs`** | **测试依赖**。仅在执行 `doCheck = true` 运行测试用例时才需要的工具。 | `[ gtest pytest ]` |

### 2. 构建生命周期属性 (Standard Phases)
Nix 的构建分为多个阶段（Phases），你可以通过这些属性覆盖默认行为。

| 属性 | 用途 | 备注 |
| :--- | :--- | :--- |
| **`src`** | **源码来源**。可以是本地路径、`fetchFromGitHub` 等。 | 必填 |
| **`patches`** | **补丁列表**。在配置前自动应用到源码的 `.patch` 文件。 | `[ ./fix-bug.patch ]` |
| **`postPatch`** | **补丁后脚本**。应用补丁后，运行一段 shell 命令（常用于 `substituteInPlace` 修改代码）。 | `substituteInPlace ...` |
| **`configureFlags`** | **配置参数**。传递给 `./configure` 或 `cmake` 的参数列表。 | `[ "--enable-feature" ]` |
| **`cmakeFlags`** | **CMake 专用参数**。如果使用了 CMake 构建系统，参数写在这里。 | `[ "-DENABLE_VNC=ON" ]` |
| **`buildFlags`** | **编译参数**。传递给 `make` 命令的参数。 | `[ "all" ]` |
| **`installFlags`** | **安装参数**。传递给 `make install` 的参数。 | `[ "DESTDIR=$(out)" ]` |
| **`doCheck`** | **是否运行测试**。布尔值，默认为 `false`。 | `true` |

### 3. 控制与路径属性 (Control & Paths)
| 属性 | 用途 |
| :--- | :--- |
| **`pname`** | 包名（Package Name）。与 `version` 组合生成 `name`。 |
| **`version`** | 版本号。 |
| **`outputs`** | **多输出设置**。默认为 `["out"]`。可以改为 `["out" "dev" "bin"]` 减小最终包体积。 |
| **`enableParallelBuilding`** | **并行编译**。设为 `true` 会让 `make` 使用 `-j$NIX_BUILD_CORES`。 |
| **`dontWrapQtApps`** | **禁用 Qt 封装**。在写 Qt 程序时常见，防止 Nix 自动对二进制进行路径封装。 |
| **`shellHook`** | **开发环境钩子**。仅在使用 `nix-shell` 或 `nix develop` 进入环境时执行的 shell 命令。 |

### 4. 关键环境变量 (Magic Variables)
在 `postInstall` 或其他脚本块中，你会经常看到这些预设变量：

*   **`$out`**: **最重要的变量**。指向该包在 `/nix/store` 中的目标安装路径。你必须把文件拷贝到这个路径下，否则包就是空的。
*   **`$src`**: 源码解压后的目录路径。
*   **`$buildInputs` / `$nativeBuildInputs`**: 包含所有依赖项路径的字符串。

### 一个典型的例子 (以 C++/Qt 项目为例)
当你看到这段代码，你就能通过上述列表快速理解它的意图：

```nix
stdenv.mkDerivation rec {
  pname = "my-cnc-app";
  version = "1.0.0";

  src = ./.;

  # 编译时用的工具
  nativeBuildInputs = [ cmake qt6.wrapQtAppsHook ]; 
  
  # 链接时用的库
  buildInputs = [ qt6.qtbase opencascade ]; 

  # 传递给 CMake 的参数
  cmakeFlags = [ "-DCMAKE_BUILD_TYPE=Release" ];

  # 安装后的额外操作：比如拷贝一些说明文档
  postInstall = ''
    mkdir -p $out/share/doc
    cp README.md $out/share/doc/
  '';

  meta = {
    description = "A CNC simulation tool";
    license = lib.licenses.mit;
  };
}
```

**重点提示：**
在读别人的 Nix 文件时，如果看到 `rec { ... }`，这意味着属性之间可以**互相引用**（比如 `name = "${pname}-${version}";`）。如果你发现某个属性不在上面的标准列表里，它通常会被 Nix 自动导出一个同名的 **Shell 环境变量**，供构建脚本内部使用。


---

## 结语

Nix 的美学在于其**一致性**。无论是描述一个简单的 `Hello World` 还是构建一个复杂的分布式集群，底层的砖块永远是那几个：**原子类型、属性集和函数**。

当你不再被 `.nix` 文件里层层嵌套的大括号吓到，而是开始寻找“这个 Set 有哪些属性”以及“这个函数解构了什么”的时候，你就已经真正踏入了 Nix 的大门。

---

> **💡 提示：** 
> *   如果你在读别人的代码时看到 `...`，说明他在追求配置的向后兼容性。
> *   如果你看到 `@`，说明他正在优雅地进行参数转发。
> *   如果你看到 `rec`，请留意那些互相耦合的变量。
```
