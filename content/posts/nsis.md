---
title: "Nsis"
date: 2024-04-20T22:18:12+08:00
draft: false
tags:
  - nsis
---

一般的编译成release之后，找个地方一扔就可以运行了。但是有些时候，我们需要在用户机器上执行个脚本，
然后还要创建几个快捷方式。搞一个安装程序，会让用户觉得更专业，并且不用再写个安装说明什么的。

## nsis

据说 [Wix](https://wixtoolset.org/) 比较专业，但是咱们没有那么专业的需求，wix看起来有点复杂。
[nsis](https://nsis.sourceforge.io/Main_Page)是另一个不错的选择，尤其当我们看完 [Tutorial](https://nsis.sourceforge.io/Simple_tutorials)，
感觉真是挺简单的。然后还是不知道从哪里下手写nsi文件。

幸好我们还有[nsis edit](https://hmne.sourceforge.net/)，点几下，填上名称、版本、图标、文件就可以生成了nsi文件了。
生成之后，使用makensis.exe来生成安装程序即可。

nsis的脚本语言，也不想花太多时间去学习，在用nsis edit生成的模板上，改改就好了，不会就google下。总结了以下几点：
- `!include "MUI.nsh"` 可以改成 `!include "MUI2.nsh"`，这样功能会更多一点
- `OutFile`后面是生成的安装程序名称
- 安装和卸载的exe图标是`!define MUI_ICON`和`!define MUI_UNICON`后面的（由于Windows缓存，修改了图标之后，生成的exe未必改了，实在不行，可以尝试重命名成别的名字，看看图标会不会变化）
- 安装时的标题栏附近的图片按如下修改，像素必须是 `150x57`
  ```nsis
  !define MUI_HEADERIMAGE
  !define MUI_HEADERIMAGE_BITMAP "header.bmp"
  !define MUI_HEADERIMAGE_RIGHT
  ```
- InstallDir后面是安装路径，如果是64位程序，应为 `"$PROGRAMFILES64\NAME"`
- 安装前卸载旧版本
  ```nsis
  Function .onInit
    ReadRegStr $R0 ${PRODUCT_UNINST_ROOT_KEY} "${PRODUCT_UNINST_KEY}" "UninstallString"
    ExecWait "$R0"
  FunctionEnd
  ```
- 递归添加文件夹`File /a /r "aaa\"`，注意aaa文件夹下面的文件会添加到输出目录，而不是aaa本身
- 添加可选文件`File /nofatal /a "aaa.ico"`
- 创建文件夹`CreateDirectory`
- 创建快捷方式`CreateShortCut`，注意快捷方式的启动路径是由`SetOutPath`来设定的
- 递归删除文件夹`RMDir /r "$INSTDIR\aaa\"`
- 仅当非空时删除文件夹`RMDir "$INSTDIR"`
- 删除文件，如果文件不存在，不会报错`Delete "$INSTDIR\uninst.exe"`
- 出错时停止安装`Abort`
- 执行程序（不弹黑框，输出到安装窗口），出错时停止安装
  ```nsis
  nsExec::ExecToLog `"$INSTDIR\aaa.exe" --args`
  Pop $0
  ${IfNot} $0 == 0
    MessageBox MB_OK "安装xxx失败"
    Abort
  ${EndIf}
  ```