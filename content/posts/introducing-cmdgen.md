---
title: "我做了个小工具：用一句话生成 Shell 命令"
slug: "introducing-cmdgen"
date: 2026-10-21T16:50:00+08:00
tags: ['cmdgen', 'CLI', 'Shell', 'AI', 'Rust']
description: "想在终端里用自然语言补命令，Warp 又老推 Agent 又要升级。现成工具功能太多，我就自己写了个 cmdgen：只生成命令，不替你执行。"
series: ['Tools']
---

偶尔会卡在一条命令的参数上。这时候我宁可说一句人话，让工具吐出一条建议，我再改改、自己按回车。不要聊天框，也不要突然变成「帮你干活的 Agent」。

于是有了 [cmdgen](https://github.com/muzhix/cmdgen)。

## 先从 Warp 碰壁说起

我本来想把 Warp 当主终端用，Free 档还宣传可以自带模型额度。想法很朴素：平时还是敲 bash / zsh；想不起命令时，用自然语言要一条建议，人还在 terminal 里。

实际用起来不太客气。输入几下就容易被判定成 Agent 会话；想走命令建议（比如 `#`），不是被赶去切 Agent，就是提示升级、额度不够。后来把 ChatGPT 的 plan 接上了，界面仍说要 upgrade——那条连接主要给 Warp 的 Agent 用，管不到我想要的「命令搜索」。免费能用的 Next Command，也只是根据上下文猜下一条，不是你随便说一句它就懂。

折腾记录在[这儿](https://chatgpt.com/share/6ad8700d-7a18-83ed-ae98-8be29cbc128f)。看完大概只会得出一个不体面但有效的结论：这条路别走了，自己写一个更省事。

## 为什么不直接用现成的

我也翻过 ShellGPT、zsh-ai-cmd、Builder.io 的 AI Shell 之类。有人建议包一层 ShellGPT 就完事。问题是它们往往还带着聊天、记忆、一整套助手能力。我只要两样：根据描述生成一条命令，以及在 zsh / bash 输入行里快捷调用。多出来的部分我不想养。

所以决定重做。语言选 Rust，做成独立命令行工具，装好就能用。Key 自己准备，默认走 DeepSeek，也能接到兼容 OpenAI 那套接口的服务。不绑死 Warp，哪个终端都行。

名字也纠结过。先叫 `cmdsg`，差点叫 `cmdask`，最后叫 **cmdgen**，意思直白：command generator。仓库简介就一句话：*AI-powered shell command generation for your terminal.*

## 用起来什么样

直接跑的时候，标准输出里只有命令本身，方便接着管道用：

![在命令行里用自然语言生成命令](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-cli.gif)

装上 shell 集成后，在输入行写人话，默认按 `Ctrl+G`。Zsh 里建议会以浅色「幽灵文字」出现，`Tab` 或右方向键接受；看着像删库跑路那种命令会标红并带警告，你没明确接受前，它不会写进输入行：

![在 Zsh 输入行里生成命令建议](https://raw.githubusercontent.com/muzhix/cmdgen/main/docs/demo-shell.gif)

刻意没做的事：闲聊、记上下文、写代码、当 Agent、帮你点工具。也不会偷偷执行生成结果——没有「直接跑」这种开关。Key 放进系统钥匙串（或权限收紧的本地文件），不写进配置文件。

## 安装和使用（很短）

macOS / Linux：

```bash
curl -fsSL https://raw.githubusercontent.com/muzhix/cmdgen/main/install.sh | bash
```

然后：

```bash
cmdgen config init
cmdgen config provider   # 选服务商，再存 API Key（输入不回显）
cmdgen "列出所有 docker 容器，包括已经停止的"
```

想在输入行里用：

```bash
cmdgen shell install zsh
source ~/.zshrc
# 写一句人话，按 Ctrl+G
```

配置细节见仓库的 [中文说明](https://github.com/muzhix/cmdgen/blob/main/README.zh-CN.md)。

## 收束一句

cmdgen 解决的是很小的烦：这条命令我一时写不出来。Warp 走不通，现成工具又太胖，我就写了个瘦的。读历史书讲究材料老实；写工具也差不多——能少做的，就别假装全能。

仓库：[github.com/muzhix/cmdgen](https://github.com/muzhix/cmdgen)
