---
title: "Try Toga"
date: 2024-07-28T13:38:34+08:00
draft: false
tags:
    - toga
    - python
    - briefcase
    - gui
---

[Toga](https://github.com/beeware/toga) 是一个跨平台的Python GUI库，
支持Android和iOS，有一些简单的控件。它不像kivy那么复杂，kivy要求我们去学习
一种新的脚本语言来写界面，如果都需要学一门新的语言，那为什么我不用web的方式来开发界面呢？
所以我一直没有想去深入了解kivy。

## Toga hello

所有的GUI程序大抵是相似的：
- 一个App对象
- 一个主窗口
- 一些菜单和工具栏按钮（command）
- 一些widget
- 一些container/box
- box和widget可以通过某些style来改变布局和样式

```python
import toga
from toga.style.pack import COLUMN, Pack


def button_handler(widget):
    print("button handler")
    for i in range(0, 10):
        print("hello", i)
        yield 1
    print("done", i)


def action0(widget):
    print("action 0")


def action1(widget):
    print("action 1")


def action2(widget):
    print("action 2")


def action3(widget):
    print("action 3")


def action5(widget):
    print("action 5")


def action6(widget):
    print("action 6")


class Tutorial2App(toga.App):
    def startup(self):
        brutus_icon = "icons/brutus"
        cricket_icon = "icons/cricket-72.png"

        data = [("root%s" % i, "value %s" % i) for i in range(1, 100)]

        left_container = toga.Table(headings=["Hello", "World"], data=data)

        right_content = toga.Box(style=Pack(direction=COLUMN, padding_top=50))

        for b in range(0, 10):
            right_content.add(
                toga.Button(
                    "Hello world %s" % b,
                    on_press=button_handler,
                    style=Pack(width=200, padding=20),
                )
            )

        right_container = toga.ScrollContainer(horizontal=False)

        right_container.content = right_content

        split = toga.SplitContainer()

        # The content of the split container can be specified as a simple list:
        #    split.content = [left_container, right_container]
        # but you can also specify "weight" with each content item, which will
        # set an initial size of the columns to make a "heavy" column wider than
        # a narrower one. In this example, the right container will be twice
        # as wide as the left one.
        split.content = [(left_container, 1), (right_container, 2)]

        # Create a "Things" menu group to contain some of the commands.
        # No explicit ordering is provided on the group, so it will appear
        # after application-level menus, but *before* the Command group.
        # Items in the Things group are not explicitly ordered either, so they
        # will default to alphabetical ordering within the group.
        things = toga.Group("Things")
        cmd0 = toga.Command(
            action0,
            text="Action 0",
            tooltip="Perform action 0",
            icon=brutus_icon,
            group=things,
        )
        cmd1 = toga.Command(
            action1,
            text="Action 1",
            tooltip="Perform action 1",
            icon=brutus_icon,
            group=things,
        )
        cmd2 = toga.Command(
            action2,
            text="Action 2",
            tooltip="Perform action 2",
            icon=toga.Icon.DEFAULT_ICON,
            group=things,
        )

        # Commands without an explicit group end up in the "Commands" group.
        # The items have an explicit ordering that overrides the default
        # alphabetical ordering
        cmd3 = toga.Command(
            action3,
            text="Action 3",
            tooltip="Perform action 3",
            shortcut=toga.Key.MOD_1 + "k",
            icon=cricket_icon,
            order=3,
        )

        # Define a submenu inside the Commands group.
        # The submenu group has an order that places it in the parent menu.
        # The items have an explicit ordering that overrides the default
        # alphabetical ordering.
        sub_menu = toga.Group("Sub Menu", parent=toga.Group.COMMANDS, order=2)
        cmd5 = toga.Command(
            action5,
            text="Action 5",
            tooltip="Perform action 5",
            order=2,
            group=sub_menu,
        )
        cmd6 = toga.Command(
            action6,
            text="Action 6",
            tooltip="Perform action 6",
            order=1,
            group=sub_menu,
        )

        def action4(widget):
            print("CALLING Action 4")
            cmd3.enabled = not cmd3.enabled

        cmd4 = toga.Command(
            action4,
            text="Action 4",
            tooltip="Perform action 4",
            icon=brutus_icon,
            order=1,
        )

        # The order in which commands are added to the app or the toolbar won't
        # alter anything. Ordering is defined by the command definitions.
        self.commands.add(cmd1, cmd0, cmd6, cmd4, cmd5, cmd3)

        self.main_window = toga.MainWindow()
        # Command 2 has not been *explicitly* added to the app. Adding it to
        # a toolbar implicitly adds it to the app.
        self.main_window.toolbar.add(cmd1, cmd3, cmd2, cmd4)
        self.main_window.content = split

        self.main_window.show()


def main():
    return Tutorial2App("Tutorial 2", "org.beeware.toga.tutorial")


if __name__ == "__main__":
    main().main_loop()
```

## Togo Android


跨平台到安卓上Android最难搞的还是打包，外加一些上网问题。Togo使用[briefcase](https://github.com/beeware/briefcase)
进行打包。

### 安装

```shell
pip install briefcase
```

### 创建一个新的工程

```shell
briefcase new
```

如果只是想测试下，可以加`--no-input`，其实就是所有的都默认。


不出意外的话，就会有一些工程没法下载，咱们可以自己手动建
`C:\Users\<your-user-name>\AppData\Local\BeeWare\briefcase\Cache\templates`
然后在其中手动clone两个工程

- `git clone github:beeware/briefcase-android-gradle-template.git`
- `git clone github:beeware/briefcase-template`

之后就能顺利完成创建新工程了。

### 打包工程

```shell
briefcase package android -p apk
```

以上命令是在所有都设置好的时候才生效，现在我们来设置。

#### Android SDK

首先我建议安装Android studio， 然后在 `Tools` => `SDK Manager` 中，下载SDK，注意必须要勾选 `SDK Tools` 中的 `Android SDK Command-line Tools`
![sdk.png](sdk.png)

然后修改环境变量，将 `ANDROID_HOME` 设置为SDK所在目录，默认为 `C:\Users\<your-user-name>\AppData\Local\Android\Sdk`


#### JDK

briefcase目前要求JDK17，可以在微软的网站上[下载](https://download.visualstudio.microsoft.com/download/pr/d049fa44-4077-433e-be41-c5532bc38925/f45cac10da60a814accf1961d673906b/microsoft-jdk-17.0.11-windows-x64.zip)，
下载完随便解压到一个地方，然后修改环境变量，将 `JAVA_HOME` 指向解压的目录即可

#### Gradle

[Gradle](https://mirrors.aliyun.com/gradle/distributions/v8.2.0/gradle-8.2-bin.zip)可以从这里下载，
但是默认不是从这里下载的，由于我在下载完成前，脚本已经从官方成功下载的这个文件，所以没研究如何手动安装gradle，也许[这里](https://www.geeksforgeeks.org/how-to-install-gradle-on-windows/)可以

但是gradle的镜像还是要设置的，将这个 [init.gradle](init.gradle) 文件放到 `C:\Users\<your-user-name>\.gradle` 中即可

#### 签名

经过以上配置，应该能成功生成apk文件，但是如果这个时候将apk文件下载到手机中，
会提示 `packageInfo is null`，表示没有签名。briefcase官方文档中关于
keytool的描述根本就没法找到，经过研究，我采用下面的方式。

打开Android Studio，如下动图，创建一个jks文件，并在其中创建一个key，请记住 `Alias` 和 `Password`
![创建一个新的key-store](new-key-store.gif)

使用[APK Tool](https://objects.githubusercontent.com/github-production-release-asset-2e65be/393104474/8b78fd29-9f6c-4926-9247-14998ac7d218?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240727%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240727T155015Z&X-Amz-Expires=300&X-Amz-Signature=2e403978f0f52da43002f3e495510a85fe76b1e5ad8128aa279e31f946d759bf&X-Amz-SignedHeaders=host&actor_id=451679&key_id=0&repo_id=393104474&response-content-disposition=attachment%3B%20filename%3DAPK.Tool.GUI.v3.3.1.2.zip&response-content-type=application%2Foctet-stream)签名，
先要将签名按Android Studio生成的jks来设置，如下图所示
![签名设置](sign-settings.png)

最后回到主页来，对生成的apk进行签名，签名无误后即可下载到手机中安装
![签名](do-sign.png)

### 结束语

我想我可以教小朋友们写自己的手机app了。

如果要处理带c扩展的库的话，还有很多问题，暂时还没有尝试，参见[这里](https://github.com/beeware/briefcase/issues/471)
