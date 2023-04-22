---
title: "体验下 slint ui （https://slint-ui.com/）"
date: 2023-04-22T13:09:00+08:00
draft: false
---

先总结下结论：这个框架目前功能还不完善，但是想法真是挺好的，如果路线不错，将来还是有希望的。

slint-ui是Qt前员工搞出来的一个新的ui，用rust写的，目前支持使用rust/c++/javascipt开发。ui使用了一个新的语言，这个语言很像 QML，描述控件的功能都可以完成，但是复杂的action还得用开发语言来写。重要的是，这个语言最后会直接生成开发语言，一起编译，而不是像QML里的javascript，还需要javascript运行时，所以速度肯定是要快一些的。不好的是，生成的代码不易读懂，基本没法调试。

作为一个有QML开发经验的工程师，用起来感觉比 egui/iced 更简单一些，相比tauri不需要掌握一大堆前端开发的东西。

以下列举下slint ui语言相对QML有哪些改进

- 支持`for`，不需要 Repeater，更直白一些
- 支持`if`，而不是自己来操作`visible`
- 支持`percent`数据类型，相对父对象的百分数，写起来更简单一些，比如 `width: 50%`
- 支持双向绑定`<=>`
- `property`声明时，支持 `private，in/out/in-out`，意义更明确，也不会意外地用错
- 支持`struct`，将数据和ui对象区分开来，意义更明确
- `function`和`callback`也是静态类型，参数和返回值均为静态类型，编译时可检查很多错误
- 一个文件中可以声明多个`struct`和`component`，不`export`的在外面是看不到的，可以更容易保护内部使用的`struct`和`component`，export的时候还可以使用不同的名称
- 全局单例声明更容易，在开发语言中访问也很方便。
- `property binding`的表达式`必须`为 `pure`，避免一些难调试的问题
- `property binding`的表达式，依赖项变化时，设置为`dirty`，但是只在evalute时才更新，我理解是不会立即更新，不像QML中
