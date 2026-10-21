---
title: "我写了一个小工具，用一句话生成终端命令"
slug: "introducing-cmdgen"
date: 2026-10-21T16:50:00+08:00
tags: ['cmdgen', 'CLI', 'Shell', 'AI', 'Rust']
description: "想在终端里用自然语言补一条命令，试过 Warp 却总被推去 Agent 或升级。现成工具功能又偏多，于是自己写了 cmdgen：只负责生成命令，不替你执行。"
series: ['Tools']
---

写程序的人大多有过这种时刻：明明知道自己要干什么，一时却想不起那条命令该怎么写。参数记混了，选项也对不上，于是去搜文档，或者打开某个「更聪明」的终端，指望它听懂人话。

我希望的用法其实很普通。平时我仍然自己敲 bash 或 zsh；只有卡壳的时候，才用一两句中文描述意图，让工具给出一条可以修改的命令建议。给我建议就够了，不要把整个窗口变成对话，也不要擅自替我执行。

为了这个需求，我做过一轮尝试，最后写成了 [cmdgen](https://github.com/muzhix/cmdgen)。

## 为什么 Warp 满足不了

起初我打算把 Warp 当作日常主终端。它的免费档宣传可以接入自己的模型服务；我手头也有 ChatGPT、Cursor、智谱等已经在用的账号和额度，希望把它们接到 Warp 里继续用。我的目标很明确：打开的是终端模式，主要仍在跑 shell 命令；遇到不会写的命令，再用自然语言描述，换回一条建议，人还留在终端里。

试用之后，情况和我想的不一样。我在终端里输入内容时，Warp 很容易自动判定这是一次 Agent 会话，界面会提示进入新的 agent conversation。即便按文档去用「命令建议」相关入口（比如用 `#`），结果也常常是二选一：要么切换到 Agent 模式，要么提示需要升级，或者说额度用完了。

后来我按说明把 ChatGPT 的套餐接进 Warp，界面依旧要求升级。仔细看下来才明白：那条连接主要是让 Warp 的 Agent 去调用 OpenAI 的模型，并不能用来做我想要的「用自然语言搜索命令」。免费可用的 Next Command 倒是有，但它更像根据当前上下文猜测下一条命令，并不是我随便说一句，它就能给出对应命令。

到这里我已经比较确定了。我要的不是更强的 Agent，而是终端里那一点命令提示；Warp 的免费功能里，并没有稳定提供这一点。继续在产品限制里试来试去意义不大，不如自己写一个只做这一件事的小工具。

## 为什么不直接用现成项目

动手之前，我也看过一些类似项目，例如 ShellGPT、zsh-ai-cmd，以及 Builder.io 的 AI Shell。有人建议说，在 ShellGPT 外包一层命令就够了，不必新开仓库。

我想了想，还是没有那么做。这些工具往往不止「生成一条命令」，还会带上聊天、记忆，以及一整套助手式能力。对我来说，真正需要的只有两块：一是根据自然语言描述生成一条 shell 命令；二是在 zsh 或 bash 的输入行里，用快捷键把建议填回来。其余能力我暂时用不上；留着会让安装和配置变复杂，也让工具的职责变得不清楚。

于是我决定自己实现。语言选了 Rust，做成一个独立的命令行程序，安装尽量简单。接口密钥由我自己准备，默认使用 DeepSeek，也可以改成任何兼容 OpenAI 对话接口的服务。它不绑定 Warp，普通终端里就能用。

名字叫 `cmdgen`，意思很直白，就是 command generator，做AI驱动的shell命令生成，即 AI-powered shell command generation for your terminal.

## 使用效果

直接在命令行里调用时，标准输出里只有生成出来的那一条命令，方便再接到别的程序后面：

![在命令行里用自然语言生成命令](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-cli.gif)

如果装上 shell 集成，就可以在输入行里写自然语言，默认快捷键是 `Ctrl+G`。在 zsh 里，建议会以浅色的预览文字出现，按 `Tab` 或右方向键接受；如果建议看起来像删除文件、结束进程这类风险较高的命令，会用红色标出来并带上警告，在你明确接受之前，它不会写进输入行：

![在 Zsh 输入行里生成命令建议](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-shell.gif)

我也刻意限制了范围。它不负责闲聊，不保存多轮对话，不生成大段代码，不当 Agent，也不替你调用一堆外部工具。生成结果不会自动执行，没有「直接跑」之类的开关。密钥放进系统钥匙串，或者权限收得很紧的本地文件，不会写进普通配置文件里。

## 安装和使用

在 macOS 或 Linux 上，可以这样安装：

```bash
curl -fsSL https://raw.githubusercontent.com/muzhix/cmdgen/main/install.sh | bash
```

然后做一次初始化，选好服务商并保存密钥：

```bash
cmdgen config init
cmdgen config provider
cmdgen "列出所有 docker 容器，包括已经停止的"
```

如果希望在输入行里直接用，再装上对应 shell 的集成：

```bash
cmdgen shell install zsh
source ~/.zshrc
```

之后在输入行写一句自然语言，按 `Ctrl+G` 即可。更细的配置说明见仓库里的 [中文 README](https://github.com/muzhix/cmdgen/blob/main/README.zh-CN.md)。
