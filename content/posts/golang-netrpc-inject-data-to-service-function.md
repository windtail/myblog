---
title: "golang net/rpc inject data to service function"
date: 2023-04-22T12:48:00+08:00
draft: false
---

在golang中，net/rpc库比较牛，只需要写函数，然后使用现成的 ServerCodec 就可以完成rpc服务了。但是有个问题，service函数的参数都是来自客户端的，如果服务器想为某个特殊的函数注入一些配置或状态参数，就不好弄了。

解决方案：

修改service函数，比如原来的参数是 `FuncArgs` 结构体，现在改成

```golang
type WithDataArgs interface {
  SetData(data Data)
  RealArgs() interface{}
}

type WithDataFuncArgs struct {
  FuncArgs
  data Data
}

func (a *WithDataFuncArgs) SetData(data Data) {
  a.data = data
}

func (a *WithDataFuncArgs) RealArgs() {
  return &a.FuncArgs
}
```

重载 ServerCodec，重写 ReadRequestBody 函数

```golang
type serverCodec struct {
  rpc.ServerCodec
}

func (s *serverCodec) ReadRequestBody(funcArgs interface{}) {
  if d, ok = funcArgs.(WithDataArgs); ok {
    d.SetData(data)
    s.ServerCode.ReadRequstBody(d.RealArgs())
  } else {
    s.ServerCode.ReadRequstBody(funcArgs)
  }
}
```

