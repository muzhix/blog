# 博客维护手册

日常维护看这里。安装、配置的完整背景见 [README.md](README.md)。

站点信息速查：

| 项 | 值 |
| --- | --- |
| 正式地址 | <https://blog.hanbd.me/> |
| 仓库 | `git@github.com:muzhix/blog.git`（分支 `main`） |
| 托管 | GitHub Pages，由 `.github/workflows/pages.yml` 自动部署 |
| Hugo | 0.166.0 extended（主题允许 0.163.0–0.166.0，不要随手升级） |
| 主题 | Blowfish，子模块锁定在 `51a361e`（v3.8.0） |
| DNS | 阿里云，`blog` CNAME 指向 `muzhix.github.io` |

## 一、写文章到发布：标准流程

```bash
# 1. 起草（模板自带 draft: true）
hugo new content/posts/my-note.md

# 2. 本地预览（含草稿）
hugo server -D
# 浏览器打开 http://localhost:1313/posts/my-note/

# 3. 定稿：检查 front matter，删掉 draft 行
#    title / slug / date / tags / description 缺一不可

# 4. 构建验证（可选但推荐，和线上一致）
hugo --minify

# 5. 发布
git add content/posts/my-note.md
git commit -m "发布：<文章标题>"
git push
```

推送后 1–2 分钟，Actions 跑完即上线。打开 <https://blog.hanbd.me/posts/my-note/> 确认。

### front matter 检查清单

- [ ] `title`：文章标题
- [ ] `slug`：英文短横线，决定网址，发布后不要再改
- [ ] `date`：**不能是未来时间**，否则 Hugo 静默不构建，页面直接缺失
- [ ] `tags`：至少一个，列表页和搜索靠它
- [ ] `description`：「痛点 + 做法 + 结果」，列表页摘要和 SEO 都用它
- [ ] `draft` 行已删除或改为 `false`
- [ ] 相关文章用同一个 `series` 值串成内链（可选）
- [ ] 配图放 `static/images/posts/<slug>/`，文中用根路径引用 `/images/posts/<slug>/xx.webp`

### 写作格式约定

- 流程图用 `{{< mermaid >}}` 短代码，不要用 ```mermaid 代码块
- 提示块用 `{{< alert >}}` 短代码
- 不要连续空行超过两行
- 示例写法参考 `content/posts/welcome.md`

## 二、检查部署状态

```bash
# 看最近几次部署
gh run list --workflow pages.yml --limit 5
# 或浏览器打开 https://github.com/muzhix/blog/actions
```

- 推送 `main` → 自动构建部署；打开 PR → 只检查构建（`hugo.yml`），不部署
- Actions 全绿但页面没更新：等 1–2 分钟再强刷（`Cmd+Shift+R`），GitHub Pages 的 CDN 有短暂缓存

## 三、常见问题排查

**推送后 Actions 红了**

进 Actions 页点开失败的那次，看日志。最常见两类：
- 构建失败：本地跑 `hugo --minify` 复现，通常是 front matter 写错或模板语法问题
- 部署失败但构建绿：多为 Pages 权限/域名配置问题，检查 Settings → Pages

**文章页 404**

按顺序查：
1. `date` 是不是未来时间（最常见）
2. `draft: true` 是不是忘了删
3. 网址里的 slug 和 front matter 是否一致
4. Actions 是否真的跑完且全绿

**本地能看，线上样式/图片坏了**

`hugo server` 和生产的差异主要在 `baseURL`。检查资源引用是不是误写了相对路径或 `localhost`。

**域名打不开**

```bash
dig blog.hanbd.me          # CNAME 应指向 muzhix.github.io
curl -I https://blog.hanbd.me/   # 应返回 200
```

DNS 对但证书报错：Settings → Pages 确认 Enforce HTTPS 还勾着，证书偶尔需要几分钟重签。

## 四、发布错了怎么回滚

- **文章内容错了**：改正再推一次即可，不用回滚
- **某次部署整体搞坏**：`git revert <坏提交>` 后推送；或 Actions 页找到上一笔成功的 `Deploy GitHub Pages`，重跑（re-run）它对应的提交
- **已推送的标签不要改**，需要修正就新打一个标签

## 五、升级 Blowfish 主题

主题锁定在 `51a361e`（v3.8.0）。上游仍活跃维护，2026-10-07 发布了 v3.9.0。升级步骤：

```bash
cd themes/blowfish
git fetch origin
git log HEAD..origin/main --oneline   # 看要进来的变更
# 也可以只跟版本标签：
# git fetch origin tag v3.9.0 && git checkout v3.9.0

# 关键：确认新提交要求的 Hugo 版段
grep -A4 '\[module\]' config.toml | grep -E 'min|max'
```

如果新版本要求的 Hugo 超过 0.166.0，先别升，或同时升级 Hugo（见下一节）。

```bash
# 回到仓库根目录，本地全量预览
cd ../..
hugo server
# 检查：首页、一篇文章、标签页、搜索、目录、Mermaid、代码复制
```

没问题后提交子模块指针：

```bash
git add themes/blowfish
git commit -m "升级 Blowfish 到 v3.9.0"
git push
```

自定义模板在 `layouts/` 里盖在主题上（`layouts/partials/` 下的 head、schema、sponsor-button 等）。主题升级后如果页面异常，优先怀疑这些覆盖文件和新版主题不兼容，对照主题源码逐个核对。

## 六、升级 Hugo

当前 0.166.0，主题允许 0.163.0–0.166.0。**升级 Hugo 通常要和主题升级绑定**：

1. 查主题 `themes/blowfish/config.toml` 的 `min`/`max`
2. 本地装新版（README「环境」一节的方法）
3. `hugo server` 全量预览
4. 同步改两个工作流的 `hugo-version`：`.github/workflows/pages.yml` 和 `hugo.yml`
5. 一次提交里带上 Hugo 版本和主题指针，避免 CI 和本地版本漂移

## 七、域名与 DNS（阿里云）

- 解析：`blog` CNAME → `muzhix.github.io`（已有，勿动）
- 证书：GitHub Pages 自动续签，不需要管
- 备案：`hanbd.me` 已在阿里云 ICP 备案，页面底部如需挂备案号，写在 `config/_default/hugo.toml` 的 `aiDisclosure` 同级位置或页脚模板里
- 换正式地址（极少发生）：同步改三处——`hugo.toml` 的 `baseURL`、`static/CNAME`、GitHub Settings → Pages 的 Custom domain

## 八、暂缓功能的开启时机

| 功能 | 位置 | 开启条件 |
| --- | --- | --- |
| 赞赏按钮 | `params.toml` `[sponsor]` | 收款码放进 `static/images/` 后改 `enable = true` |
| 「在 GitHub 上编辑」 | `params.toml` `[article]` | 仓库公开后 `showEdit = true`，`editURL` 填 `https://github.com/muzhix/blog/edit/main/content` |
| 评论 | `params.toml` `[comments]` | 选好评论后端（`layouts/partials/comments.html` 有 cf-comment 示例） |
| 访问统计 | `params.toml` 底部 | 注册 Fathom/Umami 后填入 |
| 页脚 AI 声明 | `hugo.toml` `aiDisclosure` | 写成一句话即显示在每页底部 |

## 九、仓库里那些「不动」的文件

| 文件 | 作用 | 动不动 |
| --- | --- | --- |
| `static/CNAME` | GitHub Pages 自定义域名绑定 | 域名不变就别动 |
| `static/_headers`、`static/_redirects` | Cloudflare Pages 专属，GitHub Pages **不读取** | 换托管时才生效，留着无副作用 |
| `.gitmodules` + `themes/blowfish` | 主题子模块 | 只按第五节的流程动 |
| `.github/workflows/pages.yml` | 部署流水线 | 升 Hugo 时改版本号 |
| `.github/workflows/hugo.yml` | PR 构建检查 | 同上 |

## 十、换新机器继续写

```bash
git clone --recurse-submodules git@github.com:muzhix/blog.git
cd blog
# 装 Hugo 0.166.0 extended（方法见 README「环境」一节）
hugo server
```

写完直接推送，线上自动更新。
