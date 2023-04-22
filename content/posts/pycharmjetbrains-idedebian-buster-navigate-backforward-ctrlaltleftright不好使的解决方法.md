---
title: "Pycharm（Jetbrains IDE）Debian buster Navigate Back/Forward (Ctrl+Alt+Left/Right）不好使的解决方法"
date: 2020-03-07T15:38:00+08:00
draft: false
---

Debian stretch中，Ctrl+Alt+Left/Right 可以键盘设置中去掉，但buster中却没有这个选择，忍受了几天之后，终于在[这里](https://stackoverflow.com/questions/47808160/intellij-idea-ctrlaltleft-shortcut-doesnt-work-in-ubuntu)搜索到了答案。




```
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-left "[]"
gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-right "[]"
```


如果要改回去，需要执行以下命令




```
gsettings reset org.gnome.desktop.wm.keybindings switch-to-workspace-left
gsettings reset org.gnome.desktop.wm.keybindings switch-to-workspace-right
```


当然，在设置之前，还可以Get下，以确认是这个问题




```
gsettings get org.gnome.desktop.wm.keybindings switch-to-workspace-left
```


如果返回以下内容，说明就是这个问题了




```
['Left']
```


 


