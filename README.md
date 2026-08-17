# 李不言的博客

> 桃李不言，下自成蹊。

学习笔记与技术文章。

## 技术栈

- [Astro](https://astro.build/) + [AstroPaper](https://github.com/satnaing/astro-paper) 主题
- 部署：Cloudflare Pages
- 写作：Markdown，位于 `src/content/posts/`

## 本地开发

```bash
npm install
npm run dev
```

## 写新文章

在 `src/content/posts/` 下新建 `.md` 文件：

```markdown
---
title: "文章标题"
author: "李不言"
pubDatetime: 2026-08-17T12:00:00+08:00
slug: my-post
featured: false
draft: false
tags:
  - 标签
description: "一句话描述"
---

正文内容...
```

推送到 main 分支后，Cloudflare Pages 自动构建部署。
