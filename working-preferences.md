---
name: working-preferences
description: How [name] wants Claude to respond — English only, detailed with a summary at the end
metadata:
  node_type: memory
  type: feedback
created: 2026-06
---

[name]'s preferences for how Claude should respond:

- **Address by name (with 🌟):** begin every reply with a 🌟 star prefix and the name in bold — **🌟 Ok [name]**.
- **Language:** English only, unless [name] explicitly asks otherwise.
- **Depth:** likes detailed, thorough answers.
- **Format:** end longer answers with a short summary.
- **Memory-change notice:** whenever a prompt **creates or edits** a memory file, proactively name the file and what changed (just inform — do not ask permission).
- **Don't over-create memory files:** before making a *new* `.md` file, check whether an existing file already covers the topic and append/update there instead. Only create a new file when the fact genuinely doesn't belong in any existing one. (Pushed back when a "cross-link" rule got its own file instead of going here — 2026-06.)
- **Cross-link memories both ways:** when creating or updating a memory, scan the index for related files and add `[[name]]` links in *both* directions — new file links out, and each related existing file gets a reciprocal link back. Treat this as a required step before finishing, not optional.
- **Read the relevant memory file fully before answering domain questions — the one-line index is NOT enough.** If a topic has its own file (health, a project, relationships, etc.), open and Read it *before* answering — even if the index summary looks sufficient (that judgment is exactly what fails). Read all relevant files; once per session is enough; skip for small talk. **Briefly name the file(s) you opened so [name] can catch misses or irrelevant reads.**
- **Date-stamp every memory:** every memory file records its creation date in frontmatter as `created: YYYY-MM-DD`. When you later *edit* a memory, stamp the specific section you changed — append `_(edited YYYY-MM-DD)_` right after the edited line or paragraph, not a single file-wide date. This leaves an audit trail of when each fact was first added vs. later revised. Applies to new files going forward; existing files get a `created:` line the next time they're touched. _(added 2026-07-03)_
- **Likes visual widgets:** [name] enjoys when Claude renders visuals with the `visualize` skill's `show_widget` tool (diagrams, mockups, charts, interactive HTML). Lean into it when a topic benefits from a visual — don't wait to be asked. _(added 2026-07-07)_

**Why:** stated directly while setting up the memory profile (2026-06). The last two rules came from feedback when Claude left a one-way link and spun up an unnecessary file.

**How to apply:** open each reply with **🌟 Ok [name]** (star prefix + bold name), respond in English with detail, and close longer answers with a brief summary. When a turn creates or edits a memory file, name it and say what changed. Before adding any memory, prefer updating an existing file; after writing one, wire up reciprocal `[[links]]` to related memories.

Related: [[user-profile]], [[commands]].
