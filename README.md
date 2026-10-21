# 我的博客

这是一套和 [blog.ferstar.org](https://blog.ferstar.org) 同一路子的 Hugo 站点：主题是 [Blowfish](https://blowfish.page/)（锁定在和原站相同的提交 `51a361e`），只有简体中文，放在网站根路径。文章页保留搜索、目录、系列、标签、阅读进度、Mermaid、代码复制、复制 Markdown、AI 研读和 RSS。

原来的文章、配图、幻灯片、旧地址重定向和个人 logo 都已去掉，只留两篇示例。左上角和浏览器图标用的是 Blowfish 自带的鱼标，没有再放自定义图片。

两篇示例可以留着当对照，熟悉之后删掉即可：

- `content/posts/welcome.md`、`content/posts/write-your-first-post.md`

## 发布前先改这几处

`baseURL`、`CNAME`、部署工作流都已按 <https://blog.hanbd.me/> 配好。还剩这些占位，公开之前换成你自己的：

| 要改的 | 文件 |
| --- | --- |
| 站点名 | `config/_default/hugo.toml` 的 `title`，以及 `languages.zh-cn.toml` 里的 `title` |
| 站长名字、简介 | `languages.zh-cn.toml` 的 `[params.author]` |
| 页脚版权 | `hugo.toml` 里 `copyright` |
| 导航外链 | `config/_default/menus.zh-cn.toml` |
| 安装到主屏时的名字 | `static/site.webmanifest` 的 `name` |

本地预览不用改 `baseURL`，`hugo server` 会临时用 `http://localhost:1313/`。

## 环境

主题要求 **Hugo extended**，版本 **0.163.0 到 0.166.0**。更高版本主题会直接拒绝构建。下面用 0.166.0。

macOS 的发布文件是 pkg，不是 tar.gz。下面只把二进制放到用户目录，不写入系统路径：

```bash
curl --http1.1 -fL -o /tmp/hugo.pkg \
  https://github.com/gohugoio/hugo/releases/download/v0.166.0/hugo_extended_0.166.0_darwin-universal.pkg
rm -rf /tmp/hugo-pkg
pkgutil --expand-full /tmp/hugo.pkg /tmp/hugo-pkg
mkdir -p "$HOME/.local/bin"
cp /tmp/hugo-pkg/Payload/hugo "$HOME/.local/bin/hugo"
chmod +x "$HOME/.local/bin/hugo"
export PATH="$HOME/.local/bin:$PATH"
hugo version
```

应看到 `v0.166.0` 和 `extended`。把 `$HOME/.local/bin` 写进 shell 配置，以后新开终端也能用。也可以直接打开那个 pkg 做系统安装，装完确认 `hugo version` 仍是 0.166.0 extended，而不是更新的版本。

主题在 `themes/blowfish`，是 git submodule。别人克隆时：

```bash
git clone --recurse-submodules <你的仓库地址>
```

如果已经克隆但主题目录是空的：

```bash
git submodule update --init --recursive
```

## 本地预览

在仓库根目录：

```bash
hugo server
```

- 首页：<http://localhost:1313/>
- 示例文章：<http://localhost:1313/posts/welcome/>

保存 Markdown 后页面会自动刷新。草稿默认不显示，预览草稿用：

```bash
hugo server -D
```

只生成静态文件、不启动服务器：

```bash
hugo --minify
```

产物在 `public/`。这个目录已在 `.gitignore` 里，不要提交。

## 新增文章

一篇文章就是一个 Markdown 文件：

```text
content/posts/my-note.md
```

用模板生成：

```bash
hugo new content/posts/my-note.md
```

模板会带上 `draft: true`。写完并预览通过后删掉这一行，或改成 `false`。留着 `draft: true` 再去构建，线上不会有这篇文章。

Front matter 最少要有这些字段：

```yaml
---
title: "把本地数据库迁到 Apple Container"
slug: "my-note"
date: 2026-10-11T21:00:00+08:00
tags: ['Hugo', 'Notes']
description: "数据还在旧容器里时直接删运行时会丢库；先导出再挂到新运行时，迁移后库还能打开。"
series: ['Getting Started']
---
```

约定：

- `description` 写成「痛点 + 做法 + 结果」，列表页和搜索摘要都用它。
- `date` 不能是未来时间。Hugo 默认不构建未来日期的文章，页面会悄悄缺失。
- `slug` 用英文短横线，它决定网址：`/posts/my-note/`。
- 正文不要连续空行超过两行。

示意图用 Blowfish 短代码，不要用 mermaid 代码块。下面整段原样放进文章：

````markdown
{{< mermaid >}}
flowchart LR
    A["草稿"] --> B["hugo server -D"]
    B --> C["去掉 draft"]
    C --> D["提交并推送"]
{{< /mermaid >}}
````

提示块同样用短代码：

````markdown
{{< alert >}}
这里是需要读者先看到的一句限制或更新。
{{< /alert >}}
````

两篇示例里有可以直接抄的写法。

配图放在 `static/images/posts/<slug>/`，文中用站点根路径：

```markdown
![导出完成后的目录](/images/posts/my-note/export.webp)
```

系列名会在文首串成一组内链。相关文章用同一个 `series` 值。

## 版本发布

这个博客没有单独的安装包。一次发布就是：源码进入 `main`，托管方重新构建，用新的 `public/` 替换线上文件。

日常就这样走：

```bash
git status
git add content/posts/my-note.md
git commit -m "发布：把本地数据库迁到 Apple Container"
git push origin main
```

第一次提交时把主题子模块一并放进去，不要把 `themes/blowfish` 里面的文件拆开加入：

```bash
git add .gitmodules themes/blowfish
git status
```

`themes/blowfish` 应显示为一条子模块记录（gitlink），而不是一堆主题源码。

需要留下可回溯的版本时再打标签。标签打在已经推上去、并且线上已经打开看过的那次提交上：

```bash
git tag -a v2026.10.11 -m "上线站点骨架和两篇示例"
git push origin v2026.10.11
```

GitHub 上可以基于这个标签建 Release，把这次改了什么写进说明。回滚时在仓库 Actions 页重跑上一笔成功的部署，或者 `git revert` 后再推送。不要改已经推送过的标签。

推送到 `main` 时，`.github/workflows/pages.yml` 会用 Hugo 0.166.0 extended 跑一遍 `hugo --minify` 并部署到 GitHub Pages；Pull Request 只由 `hugo.yml` 检查能否构建。

## 把网站公开

本站托管在 GitHub Pages，正式地址是 <https://blog.hanbd.me/>。部署由 `.github/workflows/pages.yml` 完成：推送到 `main` 就构建并发布。

先决条件：

- 仓库已推到 GitHub（生产分支 `main`），子模块一并推上去。
- `static/CNAME` 内容只有一行 `blog.hanbd.me`，已在仓库里。这个文件让 GitHub Pages 绑定自定义域名。
- `config/_default/hugo.toml` 的 `baseURL` 已是 `https://blog.hanbd.me/`。RSS、站点地图和分享卡片都依赖它。

一次性设置：

1. 仓库 Settings → Pages → Build and deployment → Source 选 **GitHub Actions**。
2. 仓库 Settings → Pages → Custom domain 填 `blog.hanbd.me`，勾选 **Enforce HTTPS**。证书没签发完之前勾选会失败，过一会儿再试。
3. 到阿里云 DNS 添加解析记录：

| 主机记录 | 记录类型 | 记录值 |
| --- | --- | --- |
| `blog` | CNAME | `<GitHub用户名>.github.io` |

   4. DNS 生效后（本机 `dig blog.hanbd.me` 看 CNAME 是否指向 `<GitHub用户名>.github.io`），推送到 `main` 触发部署。

以后每次推送到 `main`，GitHub Actions 会重新构建并发布。Actions 页可以看每次部署的状态，也能重跑。

注意两点：

- GitHub Pages 不读取 `static/_headers` 和 `static/_redirects`，这两份文件在 Pages 上不生效，只作为将来换托管的备份。
- 域名 `hanbd.me` 已在阿里云 ICP 备案，`blog.hanbd.me` 用 HTTPS 自定义域名没有备案阻碍。GitHub Pages 的节点在境外，大陆访问速度一般但通常可用；若以后追求大陆访问质量，可加国内 CDN（需域名备案，已满足）。

旧地址要跳到新地址时，GitHub Pages 不支持 `_redirects` 规则。用它的 Jekyll 方式：仓库根新建 `static/404.html` 做跳转页，或换用支持重定向规则的托管。

## 域名约定

全站只用 <https://blog.hanbd.me/> 这一个正式地址，`baseURL`、`static/CNAME`、GitHub Pages 的 Custom domain 三处保持一致。裸域 `hanbd.me` 是否跳转、跳到哪里，在阿里云 DNS 里配，不影响本仓库。

证书由 GitHub Pages 自动签发，Custom domain 页勾选 Enforce HTTPS 即可。DNS 刚改完会有缓存，本机可用 `dig blog.hanbd.me` 看是否已指向 `<GitHub用户名>.github.io`，再判断是证书还没好，还是记录写错。

## 可以后开的功能

这些开关现在是关的，避免空站点出现别人的链接或打不开的图片。

赞赏按钮。在 `config/_default/params.toml` 里把 `sponsor.enable` 改成 `true`，收款码放到：

```text
static/images/sponsor-wechat.webp
static/images/sponsor-alipay.webp
```

有爱发电再取消 `afdian` 那一行的注释并换成你的地址。文章标题右侧会出现「请我喝咖啡」。

「在 GitHub 上编辑」。把 `article.showEdit` 改成 `true`，`editURL` 写成 `https://github.com/<你>/<仓库>/edit/main/content`，中文稿会落到 `content/...`。

评论。`params.toml` 里 `[comments] showAndTrigger` 仍是 `false`。Blowfish 支持多种评论后端，打开前看 [主题文档](https://blowfish.page/docs/partials/#comments)。`layouts/partials/comments.html` 里留了一个 cf-comment 的例子，填 `serverUrl` 后把对应开关打开。

访问统计。`params.toml` 底部有 Fathom、Umami 的注释位。只在生产构建里加载，`hugo server` 预览时不会打点。

页脚那句 AI 声明。`hugo.toml` 里 `aiDisclosure` 现在是空的，页脚不显示。写成一句话就会出现在每页底部。

## 升级 Blowfish

界面要和现在这套保持一致时，先不要跟主题上游的 `main`。要升级时在子模块里看清变更再进：

```bash
cd themes/blowfish
git fetch origin
git log HEAD..origin/main --oneline
```

确认 Hugo 版本仍在主题 `config.toml` 的 `min` / `max` 之间，再检出新提交，回到仓库根目录预览首页、一篇文章和标签页。没问题后提交新的子模块指针。

## 目录

```text
config/_default/     站点配置、导航
content/posts/       文章
archetypes/          hugo new 用的 front matter 模板
layouts/             盖在 Blowfish 上的模板
assets/              自定义样式和脚本
static/              原样拷贝的图片、_headers、_redirects
themes/blowfish/     主题子模块
public/              构建结果，不提交
```
