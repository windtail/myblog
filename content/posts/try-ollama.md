---
title: "Try Ollama"
date: 2024-10-07T19:12:49+08:00
draft: false
tags:
  - ai
---

## Ollama介绍

[Ollama](https://ollama.com/)设计上很像是docker，有一个常驻服务，有一个客户端用来运行命令。
存储也是和docker相似，有很多的层。其中`run`、`ps`、`pull`命令更是和docker完全相同。

需要什么model就在[这里](https://ollama.com/library)找，很像docker hub，相同的model也有不同的tag，
其中一般
- `<n>b`表示参数的billion数
- `q<n>`表示量化位数
- `K`好像与`K-means`有关

`Modelfile`和`Dockerfile`相似，可以从一个model开始，构建自己的model 

## 安装

官方网站可以直接下载并安装。但是，我因为有scoop，所以就顺手直接`scoop install ollama`了，
然后就出问题了，scoop不会自动运行服务，参见[这里](https://github.com/ollama/ollama/blob/main/docs/windows.md#standalone-cli)，
简单的说，就是自己先运行下 `ollama serve`

另外，默认模型会下载到C盘，但是谁家C盘会那么大呢，需要设置 `OLLAMA_MODELS` 环境变量为模型下载文件夹，参见[这里](https://github.com/ollama/ollama/blob/main/docs/faq.md#how-do-i-set-them-to-a-different-location)

执行如下命令可以测试安装是否正确

```shell
ollama run phi3
```

最后 `/bye` 退出

## AI助手Ollama版

```python
from langchain.chains.conversation.base import ConversationChain
from langchain_community.chat_models.ollama import ChatOllama
from langchain_core.messages import SystemMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder, HumanMessagePromptTemplate
from langchain.memory import ConversationBufferMemory

def main():
    model = ChatOllama(base_url='http://localhost:11434',
                       model="qwen2.5:7b-instruct-q2_K")

    print("欢迎来到AI对话，请输入您想说的话，或`quit`退出")

    prompt = ChatPromptTemplate.from_messages([
        SystemMessage("You are a helpful assistant"),
        MessagesPlaceholder(variable_name="history"),
        HumanMessagePromptTemplate.from_template("{input}")
    ])

    memory = ConversationBufferMemory(return_messages=True)

    conversation = ConversationChain(
        llm=model,
        prompt=prompt,
        memory=memory,
    )

    while True:
        your_text = input("You: ")
        if your_text == "quit":
            break

        result = conversation.invoke({"input": your_text})

        print("AI:", result["response"])


if __name__ == '__main__':
    main()
```

- 运行前，需要下载模型 `ollama pull qwen2.5:7b-instruct-q2_K`
- `ConversationChain`中默认是`history`、`input`和`response`，如果需要更改，则需要在构造时传入额外参数
