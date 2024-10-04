---
title: "Try LLM App"
date: 2024-10-04T20:27:16+08:00
draft: false
tags:
  - ai
---

## gpt4free

```shell
pip install g4f
```

## code

```python
from typing import List, Any
import sys
from g4f.client import Client
import asyncio

if sys.platform == "win32":
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())


def fetch_ai(client: Client, messages: List[Any]) -> str:
    chunks = []

    print("AI: ", end="")

    stream = client.chat.completions.create(
        model="gpt-4",
        messages=messages,
        stream=True,
    )

    for msg in stream:
        chunk = msg.choices[0].delta.content
        if chunk:
            chunks.append(chunk)
            print(chunk, end="", flush=True)
    else:
        print("")
    return "".join(chunks)


def main():
    client = Client()

    print("欢迎来到AI对话，请输入您想说的话，或`quit`退出")

    message_history = [{"role": "system", "content": "You are a helpful assistant"}]

    while True:
        your_text = input("You: ")
        if your_text == "quit":
            break
        message_history.append({"role": "user", "content": your_text})
        ai_text = fetch_ai(client, message_history)
        message_history.append({"role": "assistant", "content": ai_text})


if __name__ == '__main__':
    main()
```

## explanation

- `gpt4free`让大家都可以访问到`openai`的模型，虽然有点慢
- `gp4free`的`Client`提供了和`openai`兼容的方法
- 我们可以用`requests`直接访问提供`openai`兼容服务，
  但是`Client`提供了更好的封装，尤其是`stream=True`的时候
- 大模型实际上并没有上下文，需要我们每次都把之前的聊天内容都通过`messages`给它
- `role`可以取三种
  1. `system`：对大模型的配置
  2. `user`：用户说的话
  3. `assistant`：大模型之前返回的话