---
title: "Python Watch File Changes"
date: 2024-04-20T21:24:33+08:00
draft: false
tags:
  - python
---

说到用Python监控文件夹的变化，大家都是推荐 [watchdog](https://pypi.org/project/watchdog/)，
但是我发现这个工具有的时候会误报，我这里误报之后会执行编译，影响比较大。

然后我就想找到一个rust的工具，自己生成一个Python绑定下。果然有这样的rust工具[notify](https://github.com/notify-rs/notify)，
更好的是还有Python绑定 [watchfiles](https://pypi.org/project/watchfiles/)

## 示例

```python
for changes in watchfiles.watch(
    watch_root_dir,
    recursive=True,
    rust_timeout=3_000, # timeout for this watch call
    yield_on_timeout=True,  # yield with empty changes if rust_timeout reached
    watch_filter=watchfiles.DefaultFilter(ignore_dirs=ignore_dirs),
):
    # do something with the changed files
    pass
```
