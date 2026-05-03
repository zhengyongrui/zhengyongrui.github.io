# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Hexo static blog site deployed to GitHub Pages. The blog is titled "zyr的笔记" (zyr's Notes) and uses the Butterfly theme.

## Commands

```bash
# Start local development server (http://localhost:4000)
npm run server

# Generate static files to ./public directory
npm run build

# Clean generated files and cache
npm run clean

# Deploy (configured but deployment type is empty in _config.yml)
npm run deploy

# Create a new post
hexo new "Post Title"

# Create a new page
hexo new page "Page Name"
```

## Architecture

- **`source/_posts/`** - Blog posts as Markdown files with YAML frontmatter
- **`scaffolds/`** - Templates for new posts/pages/drafts
- **`themes/`** - Theme directory (Butterfly is installed via npm as `hexo-theme-butterfly`)
- **`public/`** - Generated static site output (created by `hexo generate`)
- **`_config.yml`** - Main Hexo configuration
- **`_config.butterfly.yml`** - Butterfly theme-specific configuration

## Deployment

GitHub Actions workflow (`.github/workflows/pages.yml`) automatically builds and deploys to GitHub Pages on every push to `main` branch. The site URL is `https://zhengyongrui.github.io`.

## Post Frontmatter

Posts use standard Hexo frontmatter. Example:

```yaml
---
title: Post Title
date: 2026-05-03
tags:
  - tag1
  - tag2
categories:
  - category1
---
```

## Theme Configuration

Butterfly theme is configured in `_config.butterfly.yml`. Key settings include navigation menu, code highlight theme (mac style), and post metadata display options. Theme documentation: https://butterfly.js.org/