---
title: "python创建迅雷批量任务"
date: 2016-02-12T01:01:00+08:00
draft: false
---

其实不是真的创建了批量任务，而是用python创建一个文本文件，每行一个要下载的链接，然后打开迅雷，复制文本文件的内容，迅雷监测到剪切板变化，弹出下载全部链接的对话框~~


实际情况是这样的，因为用python分析网页非常，比如下载某页中的全部pdf链接




```
 1 from \_\_future\_\_ import unicode\_literals
 2 from bs4 import BeautifulSoup
 3 import requests
 4 import codecs
 5 
 6 r = requests.get('you url')
 7 s = BeautifulSoup(r.text)
 8 links = s.findall('a')
 9 pdfs = []
10 for link in links:
11     href = link.get('href')
12     if href.endswith('.pdf'):
13  pdfs.append(href)
14 
15 with open('you file', 'w', 'gb2312') as f:
16     for pdf in pdfs:
17         f.write(pdf + '\r\n')
```


 


