---
title: "GPUI Key Points"
date: 2026-01-24T12:18:47+08:00
draft: false
tags:
  - rust
  - GPUI
---

GPUI是一个使用GPU绘制的即时模式和保留模式混合的体，和很多UI框架相同，GPUI是单线程的，异步任务和后台任务需要特殊处理。

## Entity

凡是需要多处使用的地方都需要使用 Entity 包装一下，可以认为Entity和Rc差不多。不管是View，还是Model内部都是使用Entity，
当View和Model变化时，都需要调用 `cx.notify()` 来通知其他对象。


## Render, RenderOnce，IntoElement

### View

实现了Render Trait的Entity也就是View，只有View才可以单独刷新，View每次刷新时，就是调用Render::render，
这会把所有组件全部刷新一遍，并重新注册一遍回调函数。这样的好处时，可以根据状态随时改变界面，而不需要像别的UI一样，
需要管理每个组件的生命周期。

View可以套View，局部需要单独刷新（比如局部的刷新比较快，不想影响其他部分），Parent View刷新时，如果Child View并没有发生变化，
Child View就会复用上次render的结果，所以也不影响性能。

### Component

一般组件都是用RenderOnce Trait来实现，RenderOnce就是即时构造，即时用，本身并不存储状态，RenderOnce的组件需要从cx中获得状态，
或处理控件的focus等，实现了RenderOnce组件的结构体，还需要 `#[derive(IntoElement)]` 才能自动实现IntoElement

### Simple Component

更简单的组件可以直接实现IntoElement，从结构体就可以直接生成Element

### 总结

综上所述，Element可以理解为像素，任何需要显示的对象，都要转成Element才能被显示出来。
View里面会有一些对象，这些对象可以实现Render/RenderOnce/IntoElement组件中的任意一个，View每次刷新时都全部刷一次，
其中Render会有缓存，而RenderOnce和IntoElement对象都是在render中构造，并即时生成Element的。

顶层View会存储子View的Entity，会每Render都clone一下，顶层View还会存储一些状态，这些状态用于在render中构造RenderOnce和IntoElement对象。


## [gpui_component](https://github.com/longbridge/gpui-component)

长桥证券的gpui_component分两种组件
[Stateless Elements](https://longbridge.github.io/gpui-component/docs/getting-started#stateless-elements)和
[Stateful Components](https://longbridge.github.io/gpui-component/docs/getting-started#stateful-components)，
其中无状态的就是实现RenderOnce（即Component），有状态的就是实现Render（即View）

## GPUI为什么这么快

每次都重新绘制，为什么还那么快呢？因为rust+gpu很快，排版引擎也很快，然后每个组件很轻量，构造并不费什么时间，再加上没有变化时会缓存，
结果就是这样了。引用文档中的一段话：

> Most applications use the Render trait for views and RenderOnce for components, 
> only implementing Element directly when specific performance or layout requirements demand it. 
> The framework is designed so that each level builds 
> on the lower levels - Render implementations return element trees that are ultimately composed of Elements.


