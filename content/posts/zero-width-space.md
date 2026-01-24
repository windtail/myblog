---
title: "Zero Width Space"
date: 2026-01-24T13:04:24+08:00
draft: false
---

zero width space `\u200b` 是一种特殊的空白字符，
排版时，它可以告诉排版软件，这可以在这里换行，显示时又不占任何空间。

参考[这里](https://stackoverflow.com/questions/12262123/how-to-customize-qlabel-word-wrap-mode)
为了让`QLabel`可以任何位置换行，可以这么写

```python
wrap_anywhere_text = "".join(f"{t}\u200b" for t in text)
label_text = QLabel(wrap_anywhere_text, self, wordWrap=True)
```

当然，如果文本很长，然后在所有字符后面插zwsp肯定会影响性能。
