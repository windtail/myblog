---
title: "Try Mcp"
date: 2025-03-18T19:36:40+08:00
draft: false
---

最近很火的[mcp](https://modelcontextprotocol.io)感觉非常有前景，[python-sdk](https://github.com/modelcontextprotocol/python-sdk)在这里。

## 快速开始

```shell
uv init -p 3.10 test-mcp
cd test-mcp
uv add "mcp[cli]"
```

```python
# server.py
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP("Demo")


# Add an addition tool
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b


# Add a dynamic greeting resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting"""
    return f"Hello, {name}!"

```

```shell
uv run mcp dev server.py
```

这里除了配置好uv，还需要配置好node环境（推荐fnm），因为mcp inspector需要。
默认是使用stdio来运行mcp-server的，如果要使用sse，那还需要在代码的最后加上

```python
if __name__ == '__main__':
    mcp.run("sse")
```

默认的端口是 8000，这个时候要可以运行inspector，

```shell
npx @modelcontextprotocol/inspector
```

在打开的网页中，左上角，修改`Transport Type`改为`SSE`, `URL`改为`http://localhost:8000/sse` 即可
