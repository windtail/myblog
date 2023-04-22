---
title: "bash 脚本所在文件夹"
date: 2020-03-06T14:08:00+08:00
draft: false
---

来自 [stackoverflow](https://stackoverflow.com/questions/59895/how-to-get-the-source-directory-of-a-bash-script-from-within-the-script-itself)




```
DIR="$( cd "$( dirname "${BASH\_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
```


 


