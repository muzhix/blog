---
title: "Get the site skeleton running"
slug: "welcome"
date: "2026-10-10T10:00:00+08:00"
tags: ['Hugo', 'Blowfish']
description: "An empty repo does not show where pages come from; this post lines up the two content trees, the local preview, and the default Blowfish logo so you can check them in the browser."
series: ['Getting Started']
---

> I am not a native English speaker; this article was translated by AI.

This is the first of the two sample posts left in the repo. The old articles and images are gone. The theme, both languages, and the article tools — table of contents, search, Markdown copy, and AI study — are still here.

## What each folder does

Chinese posts live in `content/posts/`. English posts live in `content.en/posts/`. Use the same `slug`, `date`, and `tags` on both sides. Translate `title` and `description`. Navigation, color, and comments are in `config/_default/`. `layouts/` and `assets/` are the Blowfish overrides. You do not need to touch them to publish a post.

The fish at the top left, and the icon in the browser tab, are the logo shipped with Blowfish.

## Preview it locally

From the repository root:

```bash
hugo server
```

Chinese home is <http://localhost:1313/>. English home is <http://localhost:1313/en/>. The language switch in the header shows up only when this page has a translation.

{{< mermaid >}}
flowchart LR
    A["content/posts/welcome.md"] --> B["hugo server"]
    C["content.en/posts/welcome.md"] --> B
    B --> D["/posts/welcome/"]
    B --> E["/en/posts/welcome/"]
{{< /mermaid >}}

Save the file and the browser reloads. Once the title, the table of contents, and this diagram are on the page, write the next post.
