# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the ICLR 2026 Blogposts Track repository, a Jekyll-based website built on the al-folio template. It hosts blog post submissions for the ICLR 2026 conference blogpost track. Submissions are made via pull requests containing anonymized blog posts.

## Common Commands

### Local Development with Docker (Recommended)
```bash
docker compose pull
docker compose up
```
Website available at `http://localhost:8080`

### Local Development with Devcontainer (VSCode)
Open the repo in VSCode and accept the prompt to reopen in devcontainer. The server starts automatically on `localhost:8080`.

### Manual Jekyll Server (inside container)
```bash
bundle exec jekyll serve --future
```

### Build for Production
```bash
bundle exec jekyll build
```
Output goes to `_site/` directory.

### Install Dependencies (Legacy/Manual Setup)
```bash
bundle install
pip install jupyter
```

## Architecture

### Key Directories
- `_posts/` - Blog posts in Markdown/HTML (named `2026-04-27-[name].md`)
- `_pages/` - Static pages (about, call for posts, submission guidelines)
- `_layouts/` - Page layouts (distill layout for blog posts)
- `_includes/` - Reusable Liquid template components
- `_bibliography/` - BibTeX files for citations
- `_sass/` - SCSS stylesheets
- `assets/` - Static assets:
  - `assets/img/2026-04-27-[name]/` - Images per submission
  - `assets/html/2026-04-27-[name]/` - Interactive HTML figures
  - `assets/bibliography/2026-04-27-[name].bib` - Per-submission citations
- `bin/` - Build and deployment scripts

### Configuration
- `_config.yml` - Main Jekyll configuration with site settings, plugins, and theme options
- `Gemfile` - Ruby dependencies including Jekyll plugins
- `.pre-commit-config.yaml` - Pre-commit hooks (trailing whitespace, YAML checks)

### Blog Post Structure
Posts use the `distill` layout with YAML frontmatter:
```yaml
layout: distill
title: [Title]
description: [2-3 sentence abstract]
date: 2026-04-27
future: true
authors:
  - name: Anonymous  # For review; real names after acceptance
bibliography: 2026-04-27-[name].bib
toc:
  - name: Section 1
  - name: Section 2
```

### GitHub Actions
- `deploy.yaml` - Automatic deployment to gh-pages branch
- `filter-files.yml` - Validates PRs only modify allowed submission files
- `comment_on_error.yml` - Notifies on PR validation failures

## Submission Workflow

1. Create post in `_posts/2026-04-27-[name].md`
2. Add images to `assets/img/2026-04-27-[name]/`
3. Add citations to `assets/bibliography/2026-04-27-[name].bib`
4. Open PR with title matching the post filename
5. Automated pipeline builds and deploys to preview URL

**Critical**: Only modify files in `_posts/` and `assets/` subdirectories matching your submission name. Modifying other files will cause automatic PR rejection.

## Key Technologies
- Jekyll (static site generator)
- Liquid templating
- Jekyll Scholar (BibTeX citation handling)
- MathJax (LaTeX math rendering)
- Distill-style layouts for academic blog posts
