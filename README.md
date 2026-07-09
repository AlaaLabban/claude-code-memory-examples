# Claude Code — memory file examples

Two sanitized examples of the file-based memory [Claude Code](https://claude.com/claude-code) keeps
per user. All personal details have been replaced with generic placeholders (`[name]`, generic paths).

| File | What it is |
| --- | --- |
| [`working-preferences.md`](working-preferences.md) | A `feedback`-type memory describing how the user wants Claude to respond (tone, language, formatting, memory-handling rules). |
| [`commands.md`](commands.md) | A `reference`-type memory defining custom hashtag commands (`#disable memory`, `#share`, `#backup`, `#report`, …), including a self-contained conversation-export script. |

## Memory file format

Each memory is a Markdown file with YAML frontmatter:

```markdown
---
name: <short-kebab-case-slug>
description: <one-line summary used to decide relevance during recall>
metadata:
  type: user | feedback | project | reference
created: YYYY-MM-DD
---

<the fact. Link related memories with [[their-name]].>
```

Files are indexed in a `MEMORY.md` file (one line each) that Claude loads every session, and
the full file is read on demand when relevant.
