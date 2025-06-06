---
title: "我的跨平台工具（持续更新）"
date: 2025-01-18T22:48:37+08:00
draft: false
---

## 安装 rust

因为很多工具都是rust写的，所以需要先安装rust，参考这个[网页](https://rsproxy.cn/#getStarted)

### Linux

```shell
export RUSTUP_DIST_SERVER="https://rsproxy.cn"
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh
```

### Windows

需要先安装[vs_BuildTools](https://aka.ms/vs/17/release/vs_BuildTools.exe)，然后再使用powershell执行以下脚本（下同）

```shell
$env:RUSTUP_DIST_SERVER="https://rsproxy.cn"
$env:RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh
```

### cargo配置

将以下内容写入到 `~/.cargo/config.toml` 中

```toml
[source.crates-io]
replace-with = 'rsproxy-sparse'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true
```

rsproxy虽然很好，但是有的时候也不够快，可以安装国内镜像管理工具`crm`

```shell
cargo install crm --locked
```

```shell
crm best sparse-download
```

## 安装跨平台shell工具

- [alacritty](https://alacritty.org/)：跨平台terminal
- [nushell](https://www.nushell.sh/)：跨平台数据驱动的shell
- [zoxide](https://github.com/ajeetdsouza/zoxide)：记录常用的文件夹
- [starship](https://starship.rs/)：prompt，支持各种开发语言
- [tealdeer](https://tealdeer-rs.github.io/tealdeer/)：命令使用方法提示工具

```shell
cargo install nushell --locked
```

后面的命令都应使用nushell运行，当前的版本是 `nu 0.101.0`


```shell

mkdir ~/.config/nushell

cargo install alacritty --locked
cargo install zoxide --lpletions/cargo/cargo-completions.nu *
use ~/.nu_scripts/custom-completions/rustup/rustup-completions.nu *
use ~/.nu_scripts/custom-completions/pnpm/pnpm-completions.nu *
use ~/.nu_scripts/custom-completions/ssh/ssh-completions.nu *ocked
cargo install starship --locked
cargo install tealdeer --locked

'

$env.config.show_banner = false
$env.RUSTUP_DIST_SERVER = 'https://rsproxy.cn'
$env.RUSTUP_UPDATE_ROOT = 'https://rsproxy.cn/rustup'

' | save --append $nu.env-path


starship init nu | save -f ~/.config/nushell/starship.nu
zoxide init nushell | save -f ~/.config/nushell/zoxide.nu
git clone --depth 1 github:nushell/nu_scripts.git ~/.nu_scripts

'

source ~/.config/nushell/zoxide.nu
source ~/.config/nushell/starship.nu
use ~/.nu_scripts/custom-completions/git/git-completions.nu *
use ~/.nu_scripts/custom-completions/poetry/poetry-completions.nu *
use ~/.nu_scripts/custom-completions/cargo/cargo-completions.nu *
use ~/.nu_scripts/custom-completions/rustup/rustup-completions.nu *
use ~/.nu_scripts/custom-completions/pnpm/pnpm-completions.nu *
use ~/.nu_scripts/custom-completions/ssh/ssh-completions.nu *

use ~/.nu_scripts/modules/virtual_environments/conda.nu

' | save --append $nu.config-path


mkdir ~/.config/alacritty/themes
git clone --depth 1 github:alacritty/alacritty-theme ~/.config/alacritty/themes

$'
[general]
import = ["($env.HOME)/.config/alacritty/themes/themes/github_light.toml"]

[window]
dimensions = { columns = 120, lines = 25 }

[font]
size = 20.0

[shell]
program = "($env.HOME)/.cargo/bin/nu"

' | save --append ~/.config/alacritty/alacritty.toml

```

## 安装python开发环境

- [uv](https://docs.astral.sh/uv/)：可以完成pyenv/pipx的功能
- [poetry](https://python-poetry.org/)：poetry在管理python包时比uv更胜一筹，有关poetry使用可以参考我的另一篇[文章]({{< ref "/posts/poetry.md" >}})

```shell
cargo install --git https://github.com/astral-sh/uv uv
uv tool install poetry
```

使用uv可以管理python，不过和pyenv不同，uv是不通过源代码编译的，而是直接下载一个编译好的。

```shell
uv python list
```
 
然后 `uv python install <your python version>` 就可以了，Windows平台还需要执行下
```shell
python -m pip install pip --upgrade --force-reinstall
```
否则pip.exe不可用

## 安装js开发环境

```shell
cargo install fnm --locked

r#'
$env.FNM_NODE_DIST_MIRROR = 'https://npmmirror.com/dist'

fnm env --json | from json | load-env

$env.PATH = ($env.PATH | prepend $env.FNM_MULTISHELL_PATH | prepend $'($env.FNM_MULTISHELL_PATH)/bin' | prepend 'node-version/.bin')

if (not ($env | default false __fnm_hooked | get __fnm_hooked)) {
  $env.__fnm_hooked = true
  $env.config = ($env | default {} config).config
  $env.config = ($env.config | default {} hooks)
  $env.config = ($env.config | update hooks ($env.config.hooks | default {} env_change))
  $env.config = ($env.config | update hooks.env_change ($env.config.hooks.env_change | default [] PWD))
  $env.config = ($env.config | update hooks.env_change.PWD ($env.config.hooks.env_change.PWD | append {|_, dir|
    if (['.node-version', '.nvmrc', 'package.json'] | any {|f| $f | path exists }) {
      fnm use --silent-if-unchanged
    }
  }))
}
'# | save --append ~/.config/nushell/fnm.nu

'
source ~/.config/nushell/fnm.nu
' | save --append $nu.config-path
```

重新启动终端，执行

```shell
fnm install --lts
```

## 安装编辑器

[helix](https://helix-editor.com/) 是一个和vim很像的编辑器，支持lsp，开箱就支持很多语言，
不需要配置。对我来说，就是提示多一点，更像是emacs和vim的结合，在normal模式，按`<space>`或`:`，
就有提示，自动补全和搜索命令，不用像vim那样记那么多命令，这点和emacs很像。

```shell
git clone --depth 1 https://github.com/helix-editor/helix
cd helix
cargo install --path helix-term --locked
```

然后还需要把源代码目录中的 `runtime` 目录，拷贝到 `~/.config/helix/runtime`，
最后使用下面的命令，测试安装和配置是否正确

```shell
hx --health
```

[themes](https://github.com/CptPotato/helix-themes/releases/tag/latest)可以在这里下载，
解压之后，将build目录中的文件拷贝到 `~/.config/helix/themes`

然后在 `~/.config/helix/config.toml` 添加 `theme = "monokai"`，也可以选择别的theme，
在hx打开时，可以执行 `theme` 命令选择一个主题，并且可以实时预览。

[这里](https://kapeli.com/cheat_sheets/Helix.docset/Contents/Resources/Documents/index)是cheat-sheet
