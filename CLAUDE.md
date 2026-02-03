# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Harmonierna Wiki is a personal digital garden built on [Quartz v4](https://quartz.jzhao.xyz), a static site generator for publishing Obsidian notes. The site is deployed at https://harmonierna.github.io/wiki.

## Common Commands

```bash
# Development - build and serve locally (http://localhost:8080)
npx quartz build --serve

# Production build (outputs to public/)
npx quartz build

# Type checking and code style validation
npm run check

# Auto-format code
npm run format

# Run tests
npm test
```

## Architecture

**Content Pipeline**: Markdown files in `/content` are processed through a three-stage plugin system:
1. **Transformers** - Parse and enhance content (frontmatter, syntax highlighting, wikilinks, LaTeX)
2. **Filters** - Include/exclude content (drafts, private pages)
3. **Emitters** - Generate output (HTML pages, RSS, sitemaps)

**Key Configuration Files**:
- `quartz.config.ts` - Main config: plugins, theme colors, analytics, ignore patterns
- `quartz.layout.ts` - UI component arrangement for content and list pages

**Tech Stack**:
- TypeScript (strict mode)
- Preact for UI components
- Remark/Rehype for markdown processing
- esbuild for bundling
- KaTeX for LaTeX, Shiki for syntax highlighting

**Core Directories**:
- `/quartz/plugins` - Plugin implementations (transformers/, filters/, emitters/)
- `/quartz/components` - Preact UI components
- `/content` - User markdown content (Topics/, Sessions/)

## Code Style

- No semicolons, 100 char line width, trailing commas
- ES modules (`"type": "module"`)
- Requires Node.js v22+

## PR Guidelines

Uses Conventional Commits. PR titles must start with: `feat:`, `fix:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, `chore:`, `build:`, `ci:`, `revert:`

The project discourages entirely LLM-generated PRs without human review/refinement.
