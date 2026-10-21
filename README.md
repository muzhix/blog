# 我的博客

这是一套和 [blog.ferstar.org](https://blog.ferstar.org) 同一路子的 Hugo 站点：主题是 [Blowfish](https://blowfish.page/)（锁定在和原站相同的提交 `51a361e`），只有简体中文，放在网站根路径。文章页保留搜索、目录、系列、标签、阅读进度、Mermaid、代码复制、复制 Markdown、AI 研读和 RSS。

原来的文章、配图、幻灯片、旧地址重定向和个人 logo 都已去掉，只留两篇示例。左上角和浏览器图标用的是 Blowfish 自带的鱼标，没有再放自定义图片。

两篇示例可以留着当对照，熟悉之后删掉即可：

- `content/posts/welcome.md`、`content/posts/write-your-first-post.md`

## 发布前先改这几处

这些还是占位，公开之前换成你自己的：

| 要改的 | 文件 |
| --- | --- |
| 正式网址 | `config/_default/hugo.toml` 的 `baseURL` |
| 站点名 | 同文件的 `title`，以及 `languages.zh-cn.toml` 里的 `title` |
| 站长名字、简介 | `languages.zh-cn.toml` 的 `[params.author]` |
| 页脚版权 | `hugo.toml` 里 `copyright` |
| 导航外链 | `config/_default/menus.zh-cn.toml` |
| 安装到主屏时的名字 | `static/site.webmanifest` 的 `name` |

`baseURL` 写成 `https://你的域名/`，末尾保留斜杠。本地预览不用改它，`hugo server` 会临时用 `http://localhost:1313/`。

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

GitHub 上可以基于这个标签建 Release，把这次改了什么写进说明。回滚时在 Cloudflare Pages 或 GitHub Pages 的部署列表里重新部署上一笔成功的构建，或者 `git revert` 后再推送。不要改已经推送过的标签。

推送到 `main` 或打开 Pull Request 时，`.github/workflows/hugo.yml` 会用 Hugo 0.166.0 extended 跑一遍 `hugo --minify`。这只检查能不能构建，不会自动部署。部署在下一节选一家托管。

## 把网站公开

构建产物是静态文件，任何能托管静态站的服务都可以。下面两家和这套仓库最合拍。更推荐 Cloudflare Pages：`static/_headers` 的缓存规则和 `static/_redirects` 的跳转只有在 Cloudflare Pages 上生效。

### Cloudflare Pages

1. 把仓库推到 GitHub 或 GitLab。
2. 打开 [Cloudflare Dashboard](https://dash.cloudflare.com/) → Workers & Pages → Create → Pages → Connect to Git。
3. 选中这个仓库，生产分支用 `main`。
4. 构建设置：
   - Framework preset：Hugo
   - Build command：`hugo --minify`
   - Build output directory：`public`
5. 环境变量：
   - `HUGO_VERSION` = `0.166.0`
   - `HUGO_ENV` = `production`
6. 保存并部署。第一次成功后会得到一个 `*.pages.dev` 地址。
7. 把 `config/_default/hugo.toml` 的 `baseURL` 改成这个地址或你的正式域名，再推一次。RSS、站点地图和分享卡片都依赖 `baseURL`。

以后每次推送到 `main`，Cloudflare 会重新构建并发布。部署记录里可以预览、也可以回滚到上一版。

旧地址要跳到新地址时，在 `static/_redirects` 里一行写一条明确规则，例如：

```text
/notes/old-title    /posts/my-note/    301
```

不要写 `/post/*` 这种通配。

### GitHub Pages

GitHub Pages 不读取 `_headers` 和 `_redirects`。可以接受这一点的话，在仓库里新增 `.github/workflows/pages.yml`：

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
      - uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.166.0"
          extended: true
      - run: hugo --minify --baseURL "${{ vars.HUGO_BASEURL }}"
      - uses: actions/upload-pages-artifact@v3
        with:
          path: public
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

然后：

1. 仓库 Settings → Pages → Build and deployment → Source 选 GitHub Actions。
2. Settings → Secrets and variables → Actions → Variables，新增 `HUGO_BASEURL`。用默认域名时填 `https://<用户名>.github.io/<仓库名>/`；用自定义域名时填 `https://你的域名/`。
3. 推送到 `main`。Actions 绿了之后，站点在 Settings → Pages 显示的地址上。

项目页（`用户名.github.io/仓库名/`）必须把这个子路径写进 `baseURL`。用户页（仓库名正好是 `用户名.github.io`）的 `baseURL` 是 `https://用户名.github.io/`。

## 使用自己的域名

顺序固定：先让默认域名能打开，再改 DNS，最后把 `baseURL` 换成这个域名并重新部署。证书由托管方签发，一般几分钟到几小时。

选定一个正式地址，全站只用它。例如只用 `https://blog.example.com/`，让裸域 `example.com` 跳过来。`baseURL` 和这个正式地址保持一致，包括 `https` 和末尾斜杠。

### 域名的 DNS 在 Cloudflare

Pages 项目 → Custom domains → Set up a domain，填 `blog.example.com`。域名已经托管在同一个 Cloudflare 账号时，它会自己加上 CNAME。等证书变成 Active，再用浏览器打开。

裸域 `example.com` 同样在 Custom domains 里加一次。Cloudflare 允许在根域写 CNAME，不用再找 A 记录。

两个主机名都加上之后，在 Cloudflare 的 Rules → Redirect Rules 里把非正规地址 301 到 `baseURL` 那一个，避免同一篇文章出现两个网址。

### 域名的 DNS 在别处，网站在 Cloudflare Pages

在 Pages 的 Custom domains 里添加域名后，Cloudflare 会给出要填的 CNAME 目标，一般是 `<项目>.pages.dev`。到域名注册商处添加：

| 主机记录 | 类型 | 目标 |
| --- | --- | --- |
| `blog` | CNAME | `<项目>.pages.dev` |

根域很多注册商不允许 CNAME。那种情况只绑 `blog` 子域，或者把 DNS 服务器改成 Cloudflare 再按上一节做。

### 网站在 GitHub Pages

1. 仓库 Settings → Pages → Custom domain 填 `blog.example.com`，等它自动写入 `static/CNAME`（内容只有一行域名）。这个文件要提交。
2. 勾选 Enforce HTTPS。证书没签发完之前勾选会失败，过一会儿再试。
3. 在 DNS 里添加：

子域名：

| 主机记录 | 类型 | 目标 |
| --- | --- | --- |
| `blog` | CNAME | `<用户名>.github.io` |

根域用这四条 A 记录，有 IPv6 再加 AAAA：

| 类型 | 值 |
| --- | --- |
| A | `185.199.108.153` |
| A | `185.199.109.153` |
| A | `185.199.110.153` |
| A | `185.199.111.153` |
| AAAA | `2606:50c0:8000::153` |
| AAAA | `2606:50c0:8001::153` |
| AAAA | `2606:50c0:8002::153` |
| AAAA | `2606:50c0:8003::153` |

4. 把 `HUGO_BASEURL` 和 `hugo.toml` 的 `baseURL` 都改成 `https://blog.example.com/`，推送后再看线上的文章链接是否还带 `github.io`。

DNS 刚改完会有缓存。本机可以用 `dig blog.example.com` 看是否已经指向托管方，再判断是证书还没好，还是记录写错。

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
