# kristal-helix-config

[![license](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue)](LICENSE-APACHE) <img src="https://img.shields.io/github/repo-size/Bli-AIk/kristal-helix-config.svg"/> <img src="https://img.shields.io/github/last-commit/Bli-AIk/kristal-helix-config.svg"/> <br>
<img src="https://img.shields.io/badge/Helix-281733?style=for-the-badge" /> <img src="https://img.shields.io/badge/LuaLS-2C6DB2?style=for-the-badge" /> <img src="https://img.shields.io/badge/Kristal-3B3B3B?style=for-the-badge" />

> **注意：** 本配置及其文档为 vibe coding 产物，但已经过人工审查。
>
> 当前状态：可用于本地 Kristal mod 开发；规模不大，偏个人使用。

| English | 简体中文 |
|---------|------|
| [English](./README.md) | 简体中文 |

**kristal-helix-config** 是一份给 Kristal mod 项目使用的 [Helix](https://helix-editor.com) 编辑器本地配置。

如果你喜欢在终端里折腾、习惯用键盘驱动一切、觉得 VS Code 有点太重了，那这份配置可能会让你觉得顺手一些。它把 Kristal mod 开发中常用的东西收进一个 `.helix` 目录里——快捷键、Lua 补全、一键启动脚本。

Helix 会自动加载项目根目录下的 `.helix/config.toml` 和 `.helix/languages.toml` 作为本地覆盖配置，所以你只需要让这层目录待在 `<你的-mod>/.helix/` 即可。仓库的结构就是直接可用的 `.helix` 目录。

目前只在 Linux 上做过测试，macOS 或 Windows 终端环境可能会遇到意料之外的问题。欢迎反馈，但暂时不保证跨平台体验。

## 简介

Kristal mod 通常需要同时处理 Lua 脚本、Love2D API、Kristal 引擎代码和 mod 自己的 `mod.json`。这份配置把这些日常会碰到的东西放在一起：

* Helix 的项目级快捷键。
* Lua Language Server 的 Kristal/Love2D 补全与诊断配置。
* 一个从当前 mod 自动启动 Kristal 的终端脚本。
* 少量 Kristal mod 全局类型提示。

## 功能

* `Space-l`：在新终端里运行当前 mod。
* `Space-L`：同样运行当前 mod，但 Love 退出后保留终端窗口。
* `Space-f`：格式化当前文件。
* `Space-o` / `Space-O`：打开文件选择器。
* 自动从当前 `.helix` 所在目录推导 mod 根目录。
* 优先从 `mod.json` 的 `"id"` 读取 Kristal mod id，缺失时回退到目录名。
* 支持用 `KRISTAL_ROOT` 指定 Kristal 引擎路径。
* LuaLS 支持 Love2D 和 Kristal 引擎源码。

## 使用方法

### 复制到你的 mod 里

```bash
git clone git@github.com:Bli-AIk/kristal-helix-config.git /tmp/kristal-helix-config
rsync -av --exclude .git /tmp/kristal-helix-config/ /path/to/your-mod/.helix/
```

### 或者作为 git 子模块添加

```bash
git -C /path/to/your-mod submodule add git@github.com:Bli-AIk/kristal-helix-config.git .helix
git -C /path/to/your-mod submodule update --init --recursive
```

这样 mod 的 `.helix` 就和上游仓库绑定在一起了，更新配置只需要更新子模块引用。如果 mod 里已经有 `.helix` 目录，先移走或从 git 索引里移除，再添加子模块。

## 配置

### Kristal 引擎路径

`run-kristal-terminal.sh` 会先从当前 mod 向上寻找最近的 Kristal 检出目录，
再回退到 `KRISTAL_ROOT`。因此项目附近的引擎优先于继承来的
`KRISTAL_ROOT`。

如果你的引擎不在这些位置，在 shell 配置里设置：

```bash
export KRISTAL_ROOT="$HOME/Projects/LuaProjects/Kristal"
```

### LuaLS library 路径

这份配置对应 Kristal `0.11.0-dev`
（`f62afea63ccab02f468c24ac0d096bd8a2c9aa81`）。LuaLS 的 API 权威来源是
当前引擎检出源码。`languages.toml` 会加载：

* `${3rd}/love2d/library`
* `.helix/kristal-mod-globals.lua`
* `${env:KRISTAL_ROOT}/main.lua`
* `${env:KRISTAL_ROOT}/src`
* `~/Projects/LuaProjects/Kristal/main.lua`
* `~/Projects/LuaProjects/Kristal/src`

要让配置更可移植，可以设置：

```bash
export KRISTAL_ROOT="$HOME/Projects/LuaProjects/Kristal"
```

## 文件说明

| File | Purpose |
| --- | --- |
| `config.toml` | Helix 项目级快捷键。 |
| `languages.toml` | LuaLS、LuaJIT、Love2D/Kristal library 和诊断设置。 |
| `run-kristal-terminal.sh` | 从当前 mod 自动启动 Kristal 的脚本。 |
| `kristal-mod-globals.lua` | 给 LuaLS 用的 mod 全局类型提示。 |

## 验证方法

在仓库根目录运行：

```bash
sh -n run-kristal-terminal.sh
taplo check config.toml languages.toml
hx --health lua
```

在你的 mod 里，可以用下面的方式看脚本推导出哪个 mod id：

```bash
TERM=dumb sh -x .helix/run-kristal-terminal.sh 2>&1 | rg 'mod_root=|mod_id=|engine_root=|title='
```

正常情况下，`krisis_knightmare` 里应该显示 `mod_id=krisis_knightmare`，`deltarune-ddd-ch1` 里应该显示 `mod_id=deltarune-ddd-ch1`。

## 依赖

| Tool | Required For |
| --- | --- |
| [Helix](https://helix-editor.com) | 使用 `.helix/config.toml` 和 `.helix/languages.toml`。 |
| lua-language-server | Lua 补全、跳转、诊断。 |
| LÖVE | 运行 Kristal。 |
| Kristal | 启动并加载 mod。 |
| kitty or xterm | `Space-l` / `Space-L` 打开独立终端窗口。 |
| taplo | 可选，用于检查 TOML。 |

## 参与贡献

这个仓库主要服务个人 Kristal mod 开发习惯，所以配置偏实用，不追求通用框架。

适合提交的改动：

* 修掉复制到新 mod 后仍然依赖旧路径的问题。
* 改善 LuaLS 对 Kristal API 的识别。
* 让运行脚本支持更多常见终端。
* 补充更准确的 Kristal mod 全局类型提示。

## 许可证

本项目的许可证可从以下二者中任选其一：

* Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) 或 [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0))
* MIT license ([LICENSE-MIT](LICENSE-MIT) 或 [http://opensource.org/licenses/MIT](http://opensource.org/licenses/MIT))

随你选择。
