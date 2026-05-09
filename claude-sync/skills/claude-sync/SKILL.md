---
name: claude-sync
description: Use when migrating Claude Code memory and skills to a new device, backing up configuration, or managing the claude-sync/ directory in wlopyst.github.io
---

# Claude Sync

Manage Claude Code memory and skills across devices via the wlopyst.github.io web interface or direct repository management.

## When to Use

- Setting up Claude Code on a new device
- Backing up current memory/skills before major changes
- Sharing configuration between machines
- Reviewing or editing memory files visually

## Quick Reference

| Action | Method |
|--------|--------|
| Upload files | Web UI → Sync section → "+ Upload" |
| Edit a file | Web UI → Sync section → "Edit" button |
| Delete a file | Web UI → Sync section → "Delete" button |
| Download all | Web UI → Sync section → "Download All as ZIP" |
| Manual add | Commit to `claude-sync/memory/` or `claude-sync/skills/` |

## Repository Structure

```
claude-sync/
├── manifest.json              # Index (name, type, description, path)
├── memory/
│   ├── MEMORY.md              # Memory index
│   ├── user_*.md              # User profile
│   ├── project_*.md           # Project context
│   ├── feedback_*.md          # Workflow preferences
│   └── reference_*.md         # External resource pointers
└── skills/
    └── <skill-name>/
        └── SKILL.md           # Skill definition
```

## Memory File Format

Use Claude Code frontmatter for auto-detection:

```markdown
---
name: Short Title
description: One-line summary for indexing
type: user|project|feedback|reference
---

Content here.
```

## New Device Setup

1. Visit `https://wlopyst.github.io` → scroll to "Claude Sync"
2. Click "Download All as ZIP"
3. Extract and copy:
   - `memory/*` → `~/.claude/projects/<project>/memory/`
   - `skills/*` → `~/.claude/plugins/` or `~/.claude/skills/`
4. Launch Claude Code — memory auto-loads

## GitHub Token Setup

The web UI requires a GitHub Personal Access Token with `repo` scope:
1. Generate at GitHub → Settings → Developer Settings → Personal Access Tokens
2. Enter in the Sync section → click "Save"
3. Token stored in browser localStorage only (never committed to repo)

## Common Mistakes

- **Missing frontmatter:** Files without `---` frontblock won't appear in manifest metadata
- **Wrong directory:** Memory goes in `memory/`, skills in `skills/<name>/`
- **Stale manifest:** After manual edits, update `manifest.json` or use the web UI
