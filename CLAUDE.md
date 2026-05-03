# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal blog built with Astro v2 using the "Creek" theme, deployed to Netlify. Blog name: "Fanoy's Blog" (subtitle: 积跬步以至千里). Site URL: `https://fanoy-blog.netlify.app/`.

## Commands

- `yarn dev` — Start dev server
- `yarn build` — Production build (runs `astro build` then `node ./scripts/search/prepare-index.js`)
- `yarn preview` — Preview production build locally

Package manager is **yarn** (not npm/pnpm, despite `.npmrc`).

## Architecture

**Framework**: Astro v2 (static site generator). Pages use `.astro` components with optional TypeScript frontmatter. No client-side JS framework.

**Styling**: Tailwind CSS v3 + SCSS. Main entry: `src/styles/main.scss` (imports global, variables, content, site, custom). Tailwind config uses custom colors (`black: #12151E`, `hot-pink: #fd2d78`) and fonts (Londrina Solid for display, Poppins for body). Tailwind is set to `important: true`.

**Blog Posts**: Markdown files in `src/pages/posts/*.md`. Each post must have this frontmatter:
```yaml
---
title: "Post Title"
pubDate: "YYYY-MM-DD"
description: "Description"
hero: "/images/image.jpg"
tags: ["tag"]
layout: "../../layouts/BlogPostLayout.astro"
---
```
Posts are auto-routed by filename (e.g., `penv.md` → `/posts/penv`). Post images live in `public/images/`.

**Routing & Pages**:
- `src/pages/index.astro` — Homepage, displays all posts sorted by `pubDate` descending in a card grid
- `src/pages/posts/[...page].astro` — Paginated post listing (21 per page)
- `src/pages/search.astro` — Client-side search page
- `src/pages/rss.xml.js` — RSS feed via `@astrojs/rss`
- `src/pages/404.astro` — Custom 404

**Layouts**:
- `BlogPostLayout.astro` — Wraps individual blog posts with Nav + BlogPost component
- `BlogPost.astro` — Post rendering component (title, hero image, optional YouTube embed, content slot, email signup)

**Search**: Lunr.js + Mark.js. The postbuild script (`scripts/search/prepare-index.js`) reads all post markdown files, extracts frontmatter + content via gray-matter, and writes `public/search-index.json`. The `SearchInput` component consumes this index at runtime.

**Integrations**: `@astrojs/sitemap` (configured in `astro.config.mjs`), Google Analytics (in BaseHead).

## Deployment

Netlify builds with `yarn build` and publishes from `dist/`. Config in `netlify.toml`. Dependabot checks npm dependencies weekly.
