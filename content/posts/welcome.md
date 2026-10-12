---
title: "站点骨架先跑起来"
slug: "welcome"
date: "2026-10-10T10:00:00+08:00"
tags: ['Hugo', 'Blowfish']
description: "空仓库看不出页面从哪来；这篇把中英目录、本地预览和 Blowfish 默认 logo 对上，浏览器里能直接核对。"
series: ['Getting Started']
---

这是留下来的两篇示例里的第一篇。原来的文章和配图都清掉了，主题、双语，以及文章页上的目录、搜索、复制 Markdown、AI 研读还在。

## 文件各管一块

中文稿在 `content/posts/`，英文稿在 `content.en/posts/`。两边的 `slug`、`date`、`tags` 写成一样，`title` 和 `description` 按语言写。导航、配色、评论开关在 `config/_default/`。`layouts/` 和 `assets/` 是在 Blowfish 上改过的页面，发文章时不用动。

左上角那条鱼，以及浏览器标签上的图标，用的是主题自带的 Blowfish logo。

## 本地先看一眼

在仓库根目录执行：

```bash
hugo server
```

中文首页是 <http://localhost:1313/>，英文是 <http://localhost:1313/en/>。右上角的语言切换只在这篇有英文对照时出现。

{{< mermaid >}}
flowchart LR
    A["content/posts/welcome.md"] --> B["hugo server"]
    C["content.en/posts/welcome.md"] --> B
    B --> D["/posts/welcome/"]
    B --> E["/en/posts/welcome/"]
{{< /mermaid >}}

改完保存，浏览器会自己刷新。标题、目录和这张图都在，就可以写下一篇。
