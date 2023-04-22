---
title: "博客搬家"
date: 2023-04-22T21:13:38+08:00
draft: false
---

首先要感谢博客园多年来提供的博客平台，不过未找到商业价值，确实很难生成下去。

# 转换原博客为markdown

- 首先在博客园后台执行备份，备份成功后下载sqlite文件
- 执行以下python脚本，将所有博客转换为markdown，并将拷贝到hugo网站的posts目录

```python
import datetime
import sqlite3
import argparse
import sys
from pathlib import Path
import markdownify
import unicodedata
import re


def slugify(value, allow_unicode=False):
    """
    Taken from https://github.com/django/django/blob/master/django/utils/text.py
    Convert to ASCII if 'allow_unicode' is False. Convert spaces or repeated
    dashes to single dashes. Remove characters that aren't alphanumerics,
    underscores, or hyphens. Convert to lowercase. Also strip leading and
    trailing whitespace, dashes, and underscores.
    """
    value = str(value)
    if allow_unicode:
        value = unicodedata.normalize('NFKC', value)
    else:
        value = unicodedata.normalize('NFKD', value).encode('ascii', 'ignore').decode('ascii')
    value = re.sub(r'[^\w\s-]', '', value.lower())
    return re.sub(r'[-\s]+', '-', value).strip('-_')


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument("-o", "--output", required=True, help="output directory")
    parser.add_argument("database", help="source database file path")
    args = parser.parse_args()

    db = Path(args.database)
    if not db.is_file():
        print(f"{db} is not a file", file=sys.stderr)
        exit(1)

    output_directory = Path(args.output)
    if output_directory.exists() and not output_directory.is_dir():
        print(f"{output_directory} is not a directory", file=sys.stderr)
        exit(1)
    output_directory.mkdir(exist_ok=True, parents=True)

    conn = sqlite3.connect(db)
    try:
        c = conn.cursor()
        c.execute(r'SELECT Title,DateAdded,Body,IsMarkdown FROM blog_Content;')
        while True:
            record = c.fetchone()
            if record is None:
                break

            title, date_str, body, is_markdown = record

            file_path = output_directory / (slugify(title, allow_unicode=True) + ".md")
            if file_path.exists():
                name = file_path.name
                i = 0
                while file_path.exists():
                    file_path = file_path.with_name(f"{name}-{i}")
                    i += 1

            date = datetime.datetime.strptime(date_str, "%Y-%m-%d %H:%M:%S")
            date = date.replace(tzinfo=datetime.timezone(datetime.timedelta(hours=8)))

            if not is_markdown:
                body = markdownify.markdownify(body)

            content = f"""---
title: "{title}"
date: {date.isoformat()}
draft: false
---

{body}
"""

            with open(file_path, "wt", encoding="utf-8") as f:
                f.write(content)
    finally:
        conn.close()
```



# 生成Hugo网站

参考 Hugo [LoveIt](https://hugoloveit.com/zh-cn/theme-documentation-basics/)的文档

# 发布到github pages

参考 Hugo [官方文档](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
实际上会比官方文档更简单，因为github本身就支持Hugo，直接选下，然后commit就行
