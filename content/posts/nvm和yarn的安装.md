---
title: "nvm和yarn的安装"
date: 2022-06-11T20:50:00+08:00
draft: false
---


我从来就没有搞过前端，不过最近想了解下tauri，所以接触了下，先记录下如何安装环境。

## nvm

大家都说nvm好，我感觉类似pyenv，安装方式参见

https://gitee.com/RubyKids/nvm-cn/tree/master

```shell
bash -c "$(curl -fsSL https://gitee.com/RubyKids/nvm-cn/raw/master/install.sh)"
```

也可以参考 https://gitee.com/mirrors/nvm 参数manual安装，把git仓库改成gitee即可，否则真是安装不了，推荐第一种方式，连国内镜像都设置好了。

国内镜像是 `export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node`

## node

装好nvm之后，需要安装node，命令行是

```shell
nvm install --lts node
```

## yarn

以前同事用npm，但是感觉lock不了依赖，后来改成了yarn，不知道npm有没有改，反正我继续用yarn就好了。但是安装yarn之前还是要先安装npm

```shell
npm install -g cnpm --registry=https://registry.npmmirror.com
cnpm install -g yarn
cnpm install -g yrm
```

yrm可以用来测试或设置npm和yarn的镜像，
```shell
yrm ls
yrm test taobao
yrm use taobao
```

非常重要一点，需要将 ``$HOME/.yarn/bin` 添加到PATH环境变量中

