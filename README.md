# Eden 的技术笔记

基于 [Astro Cactus](https://github.com/chrismwilliams/astro-theme-cactus) 重建的静态博客。

## 本地开发

```bash
pnpm install
pnpm dev
```

新文章放在 `content/posts/`，使用 Markdown 并提供 `title`、`description`、`publishDate` 和 `tags` frontmatter。

## 发布

推送到 `master` 后，`.github/workflows/deploy-pages.yml` 会构建 `dist/` 并发布到 GitHub Pages。首次启用时，请在仓库 **Settings → Pages → Build and deployment → Source** 选择 **GitHub Actions**。
