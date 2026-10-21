---
title: "为什么我做了 cmdgen：只生成命令的终端小工具"
slug: "introducing-cmdgen"
date: 2026-10-21T16:50:00+08:00
tags: ['cmdgen', 'CLI', 'Shell', 'AI', 'BYOK', 'Rust']
description: "想用 Warp 的自然语言命令搜索却卡在付费与 Agent 模式；现成替代又太重。于是做了 BYOK 的 cmdgen：一句话变成当前 Shell 的命令，不自动执行。"
series: ['Tools']
---

想在终端里用自然语言补一条命令，但不要被拽进 Agent 对话，也不想把工作流绑死在某一个终端产品上——这就是我做 [cmdgen](https://github.com/muzhix/cmdgen) 的起点。

## 起因：Warp 的「AI Command Search」用不上

日常主要还是在 Bash / Zsh 里敲命令。偶尔记不清参数时，希望能用一句话描述意图，得到一条可编辑的建议命令，**仍然停在 terminal 模式**。

我试过 Warp：打开 terminal 后，它很容易把会话切到 Agent；想走「自然语言 → 命令建议」时，又会碰到升级付费墙。后来按文档配了 ChatGPT / BYOK，界面仍提示需要 upgrade——也就是说，我真正想要的那条路径，并没有稳定落在 free / 自带 Key 的可用范围内。

那段摸索过程记在[这份对话](https://chatgpt.com/share/6ad8700d-7a18-83ed-ae98-8be29cbc128f)里。结论很简单：与其继续跟产品边界较劲，不如自己做一个**只做一件事**的工具。

## 现成方案不合适，就自己写

中间也看过几个「自然语言生成 Shell 命令」的实现。多数更偏助手或 Agent：对话、记忆、工具调用、读上下文……能力多了，边界也糊了。我想要的其实很窄：

- 输入一句描述，输出**一条**命令
- Key 自己带（BYOK），默认可用 DeepSeek，也能接 OpenAI 兼容接口
- 不自动执行；人看过、改过，再自己按 Enter
- 尽量不读历史、环境变量、目录文件，请求面要小

名字从早期的 cmdask 改成了 **cmdgen**：更直白地表示 generate command。项目一句话描述是：*AI-powered shell command generation for your terminal.*

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
