---
title: "新增一篇文章"
slug: "write-your-first-post"
date: "2026-10-11T10:00:00+08:00"
tags: ['Hugo', 'Writing']
description: "Front matter 漏写时构建往往仍然成功，列表里却没有摘要或系列；按这个顺序补中英两份稿，归档里才会成对出现。"
series: ['Getting Started']
---

上一篇把目录对上了。这篇只讲发文章时容易漏的几件事。更完整的步骤在仓库根目录的 `README.md`。

{{< alert >}}
`date` 写成未来时间的话，Hugo 默认不构建这篇文章，线上会像没写过一样。用现在或更早的时间。
{{< /alert >}}

## 从模板拉一份稿

```bash
hugo new content/posts/my-note.md
hugo new content.en/posts/my-note.md
```

模板带 `draft: true`。预览草稿要加 `-D`：

```bash
hugo server -D
```

准备公开时删掉 `draft`，或者改成 `false`。

## 两边必须对齐的字段

| 字段 | 中文 | 英文 |
| --- | --- | --- |
| slug | 相同 | 相同 |
| date | 相同 | 相同 |
| tags | 相同 | 相同 |
| series | 相同，可省略 | 相同，可省略 |
| title | 中文标题 | 英文标题 |
| description | 痛点、做法、结果 | 对应的英文 |

英文稿正文第一行留这句：

> I am not a native English speaker; this article was translated by AI.

正文里的命令、表格、Mermaid 不要改成另一套内容。配图放到 `static/images/posts/my-note/`，文中这样引用：

```markdown
![说明](/images/posts/my-note/shot.webp)
```

写完分别打开中文页和英文页，再提交。
