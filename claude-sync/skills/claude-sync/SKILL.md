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

## Current Inventory (2026-05-09)

**Memory (10 files):**
| ID | Name | Type |
|----|------|------|
| MEMORY | Memory Index | index |
| user_student | Student Info | user |
| project_network_hw | Network HW Progress | project |
| project_claude_sync | Claude Sync Feature | project |
| reference_network_course | Course Resources | reference |
| reference_claude_sync_skill | Claude Sync Skill Location | reference |
| feedback_ml_homework_workflow | ML Homework Workflow | feedback |
| feedback_seminar_report_workflow | Seminar Report Workflow | feedback |
| feedback_final_design_workflow | Final Design Workflow | feedback |
| feedback_subagent_workflow | Subagent Workflow | feedback |

**Skills (5 skills):**
| ID | Description |
|----|-------------|
| claude-sync | 跨设备迁移 Claude Code 配置 |
| course-final-design | 课程期末设计报告自动化 |
| guizang-ppt-skill | 电子杂志风格网页 PPT |
| ml-homework | 机器学习作业自动化 |
| seminar-report | 文献研讨报告自动化 |

## Pushing via API (when HTTPS blocked)

When `github.com` HTTPS is blocked but `api.github.com` works, use PowerShell to push:

```powershell
# See /tmp/push2.ps1 for full script
# Flow: create blobs → build tree → create commit → update ref
```

Key: requires GitHub PAT with `repo` scope.

## Common Mistakes

- **Missing frontmatter:** Files without `---` frontblock won't appear in manifest metadata
- **Wrong directory:** Memory goes in `memory/`, skills in `skills/<name>/`
- **Stale manifest:** After manual edits, update `manifest.json` or use the web UI
