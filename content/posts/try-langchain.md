---
title: "Try Langchain"
date: 2024-10-06T19:55:42+08:00
draft: false
tags:
  - ai
---

刚了解下[langchain](https://github.com/langchain-ai/langchain)，简单写个 [AI助手]({{< ref "/posts/download-hugging-face-model.md" >}})的`langchain`版本

## Code

### 安装依赖

```shell
pip install langchain langchain-community pyjwt python-dotenv
```

### 申请大模型帐号

到[这里](https://bigmodel.cn/)申请一个智谱AI的帐号，注册就送一些试用的，
新建一个`.env`文件，将api的key放进去
```text
ZHIPUAI_API_KEY=<your zhipuai key here>
```

### Python代码

```python
from langchain_community.chat_models import ChatZhipuAI
from langchain_core.messages import SystemMessage, HumanMessage
from dotenv import load_dotenv

load_dotenv()

def main():
    ai = ChatZhipuAI(model="glm-4-plus", temperature=0.05)

    print("欢迎来到AI对话，请输入您想说的话，或`quit`退出")

    message_history = [SystemMessage("You are a helpful assistant")]

    while True:
        your_text = input("You: ")
        if your_text == "quit":
            break
        your_msg = HumanMessage(your_text)

        message_history.append(your_msg)

        ai_msg = ai.invoke(message_history)
        print("AI:", ai_msg.content)

        message_history.append(ai_msg)


if __name__ == '__main__':
    main()
```

可以看出，咱们不再需要自己写`role`的，直接用 `SystemMessage, AIMessage, HumanMessage` 
就可以了。当然`langchain`重要的是`chain`，继续了解去了
