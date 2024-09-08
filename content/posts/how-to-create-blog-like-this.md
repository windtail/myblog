---
title: "How to Create Blog Like This"
date: 2024-09-08T18:27:03+08:00
draft: false
---

## git仓库准备

- 下载[我的博客最新版](https://github.com/windtail/myblog/archive/refs/heads/master.zip)
- 解压后，到代码根目录执行 `git init`
- 删除 `themes/LoveIt` 文件夹
- `git submodule add https://github.com/dillonzq/LoveIt.git themes/LoveIt`
- 删除 `content/posts` 中的所有子文件和子文件夹，但要保留 `content/posts` 文件
- 修改 `config.yaml`，主要是 baseURL、author、title相关
- 提交所有文件到 `master` 分支，并 `push` 到 `github`  （如果不是master分支，则需要更改 .github/workflows/hugo.yml中的`branches: ["master"]`）

## github设置

到刚push的github仓库的 `Settings`（上面最右边）=> `Pages`（左边Security上面），
使能Pages，然后在 `Build and deployment` => `Source` 中选择 `Github Actions`

## 创建新的文章

下载 [hugo extended 0.108.0](https://github.com/gohugoio/hugo/releases?q=0.108&expanded=true) 版本，
注意最好是这个版本，因为hugo经常不兼容的。下载完成后放到搜索路径中去就可以了。

`hugo new posts/my-first-blog.md` 会创建 `content/posts/my-first-blog.md` 并把时间初始化好，
注意写好后，要将 draft 设置为 false，否则不会发布。本地使用 `hugo server -D` 测试草稿是否正确。

如果markdown中包含图片或附件，则应创建一个文件夹，例如 my-first-blog，然后在这个文件夹中创建 index.md，
相关图片和附件也放到这个文件夹，index.md中可以按markdown格式引用这些图片或附件。
