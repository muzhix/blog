---
title: "为什么我做了 cmdgen：只生成命令的终端小工具"
slug: "introducing-cmdgen"
date: 2026-10-21T16:50:00+08:00
tags: ['cmdgen', 'CLI', 'Shell', 'AI', 'BYOK', 'Rust']
description: "想用 Warp 做自然语言命令建议，却卡在 Agent 自动切换与付费墙；现成方案又太重。于是做了 BYOK 的 cmdgen：一句话变成当前 Shell 的命令，不自动执行。"
series: ['Tools']
---

想在终端里用自然语言补一条命令，但不要被拽进 Agent 对话，也不想把工作流绑死在某一个终端产品上——这就是我做 [cmdgen](https://github.com/muzhix/cmdgen) 的起点。

## 起因：Warp 的命令建议用不上

日常主要还是在 Bash / Zsh 里敲命令。偶尔记不清参数时，希望能用一句话描述意图，得到一条可编辑的建议命令，**仍然停在 terminal 模式**。

我试过把 Warp 当主终端，并打算用 Free 档的 Bring your own AI，把已有的 ChatGPT、Cursor、智谱等额度接进去。真实体验却对不上：

- 开着 terminal 输入时，很容易被自动识别成 Agent（界面会提示进入 agent conversation）
- 按文档走「自然语言 → 命令建议」（例如 `#`）时，要么被推去切 Agent，要么提示 upgrade / out of credits
- 接好 ChatGPT plan 之后仍提示 upgrade：那条连接主要给 Warp Agent 用 OpenAI 模型，**覆盖不到**我想要的命令建议
- Free 里可用的 Next Command 只是根据上下文猜下一条，不是任意自然语言生成

那段摸索过程记在[这份对话](https://chatgpt.com/share/6ad8700d-7a18-83ed-ae98-8be29cbc128f)里。结论很简单：Warp 免费路径撑不起「terminal 里的命令搜索」，与其继续跟产品边界较劲，不如自己做一个**只做一件事**的工具。

## 现成方案不合适，就自己写

中间也看过 ShellGPT、zsh-ai-cmd、Builder.io AI Shell 一类实现。有人建议直接包一层 ShellGPT 就够；但我真正要的只有两块——**生成一条 shell 命令**，以及 **zsh/bash 输入行集成**——聊天、记忆、Agent、工具调用都不想要。功能一多，边界就糊，安装和心智成本也跟着涨。

于是决定用更现代的栈重做：Rust 独立 CLI，安装尽量简单；BYOK，默认 DeepSeek，并兼容 OpenAI Chat Completions 接口。工具不绑定 Warp，任意终端都能用。

名字也改过几轮：先叫 `cmdsg`，中间考虑过 `cmdask`，最后定成 **cmdgen**（Command Generator），项目描述定为：*AI-powered shell command generation for your terminal.*

## 做出来长什么样

直接调用时，标准输出里**只有命令**，方便管道接走：

![在 CLI 里用自然语言生成命令](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-cli.gif)

装上 Shell 集成后，在输入行写自然语言，默认按 `Ctrl+G`。Zsh 会把建议显示成 ghost text，`Tab` 或右方向键接受；带破坏性倾向的建议会标红并带警告，接受前不会写进输入行：

![在 Zsh 输入行里生成命令建议](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-shell.gif)

它**不做**：聊天、多轮记忆、代码生成、Agent、工具调用 / MCP。也不会 `eval` 结果，更没有 `--execute`。

技术上是 Rust 写的独立 CLI，macOS / Linux 有预编译包；Key 进系统钥匙串或权限收紧的凭据文件，不进 `config.toml`。

## 怎么装、怎么用（简述）

安装（macOS / Linux）：

```bash
curl -fsSL https://raw.githubusercontent.com/muzhix/cmdgen/main/install.sh | bash
```

快速开始：

```bash
cmdgen config init
cmdgen config provider   # 选 provider，再存 API Key（不回显）
cmdgen "列出所有 docker 容器，包括已经停止的"
```

要在输入行里用：

```bash
cmdgen shell install zsh
source ~/.zshrc
# 然后输入自然语言，按 Ctrl+G
```

更细的配置、provider、诊断见仓库 [README（中文）](https://github.com/muzhix/cmdgen/blob/main/README.zh-CN.md)。

## 小结

cmdgen 不是又一个「终端里的万能 AI」，而是把「不会写的那条命令」补出来、交给你确认。Warp 那条路走不通、现成工具又太重时，自研一个边界清楚的小工具，反而更省事。

仓库：[github.com/muzhix/cmdgen](https://github.com/muzhix/cmdgen)
