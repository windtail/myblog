---
title: "Appimage Desktop Entry"
date: 2024-10-20T22:24:02+08:00
draft: false
tags:
  - ubuntu
  - appimage
  - desktop
---

```shell
cargo install appimanager

appimanager add -n <name> -i <icon> <app-image-file>
```

如果还想要用到图标，可以把app-image-file先解压了

```shell
 ./<app-image-file> --appimage-extract
```

这会在app-image-file旁边建一个`squashfs-root`文件夹，文件内容都解压到这里了
