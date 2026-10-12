---
title: "Add a new post"
slug: "write-your-first-post"
date: "2026-10-11T10:00:00+08:00"
tags: ['Hugo', 'Writing']
description: "A build can succeed while a missing front-matter field drops the summary or the series; follow this order for both languages and the archive lists them as a pair."
series: ['Getting Started']
---

> I am not a native English speaker; this article was translated by AI.

The previous post mapped the folders. This one covers the fields that are easy to skip. The full procedure is in `README.md` at the repository root.

{{< alert >}}
If `date` is in the future, Hugo skips the page by default and the live site looks as if the post was never written. Use the current time or an earlier one.
{{< /alert >}}

## Start from the archetype

```bash
hugo new content/posts/my-note.md
hugo new content.en/posts/my-note.md
```

The archetype sets `draft: true`. Drafts show up only when you pass `-D`:

```bash
hugo server -D
```

Remove `draft`, or set it to `false`, when the post is ready to publish.

## Fields that must match

| Field | Chinese | English |
| --- | --- | --- |
| slug | same | same |
| date | same | same |
| tags | same | same |
| series | same, optional | same, optional |
| title | Chinese title | English title |
| description | problem, approach, result | the English equivalent |

Put this line at the top of every English post:

> I am not a native English speaker; this article was translated by AI.

Keep commands, tables, and Mermaid diagrams the same on both sides. Put images in `static/images/posts/my-note/` and refer to them like this:

```markdown
![caption](/images/posts/my-note/shot.webp)
```

Open the Chinese page and the English page, then commit.
