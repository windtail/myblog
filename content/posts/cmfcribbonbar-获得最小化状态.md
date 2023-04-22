---
title: "CMFCRibbonBar 获得最小化状态"
date: 2011-11-16T21:53:00+08:00
draft: false
---

  




Ribbon工具条的有个最小化的功能，我做了个按钮，想根据Ribbon是否最小化，改变下状态，可是怎么获得这个状态呢。


CMFCRibbonBar::ToggleMimimizeState()只能切换正常和最小化状态，按正常思维应该是有个类似IsMinimized()这种函数，


但是，不好意思，没有！找了半天，有这么一个函数：


GetHideFlags，返回值：


* 0：正常
* 1（AFX\_RIBBONBAR\_HIDE\_ELEMENTS）：最小化（只有类别按钮可见）
* 2（AFX\_RIBBONBAR\_HIDE\_ALL）：完全隐藏

