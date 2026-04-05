# Project Configuration for AI Assistants

## Common Commands

- **Dev server**: `pnpm dev`
- **Build**: `pnpm build`
- **Preview**: `pnpm preview`
- **Type check**: `pnpm typecheck`

## Notes Directory

- Notes are stored in `src/content/notes/`
- Create new notes directly in this directory (NOT in `content/` root)

## Frontmatter

When creating a new .md note file, include this frontmatter:

```
---
title: ""
description: ""
tags: []
updated: "YYYY-MM-DD"
coAuthor: "opencode"
---
```

**Note**: Do NOT add a `sections` array. The sidebar uses headings extracted automatically from `render()` in `[id].astro`.

## Heading Guidelines

When writing notes in `src/content/notes/`:

1. **Use "And" instead of "&"** - e.g., "Create And Manage Tables" not "Create & Manage Tables"
2. **No parenthetical subtitles** - e.g., "File Explorer" not "File Explorer (NeoTree)"
3. **No numbered prefixes** - e.g., "Basic Commands" not "1. Basic Commands"
4. **Keep sections short and descriptive** - use title case for each word

## Git Workflow

- When creating/updating .md files in `src/content/notes/`, commit and push them together in one commit
- Don't include other non-.md files in the commit

## Preferences

- Be concise - answer in 1-3 sentences
- Don't add comments to code unless asked
- Don't create new folders unless explicitly requested
