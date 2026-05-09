# Claude Sync — Design Spec

## Overview

Add a "Claude Sync" section to the wlopyst.github.io portfolio site. This feature allows users to upload, manage, edit, and download Claude Code memory files and skills via a web interface. Data is stored in the GitHub repository and synced via GitHub API.

## Goals

1. Manage Claude Code memory files (`~/.claude/projects/xxx/memory/`)
2. Manage Claude Code skills files (`~/.claude/plugins/`)
3. Support custom (non-Claude-Code) configuration files
4. Download all data as a zip with proper directory structure for new device setup
5. Inline markdown editor for file content
6. Pure frontend — no backend server required

## Architecture

### Storage Structure

Repository directory:
```
claude-sync/
├── manifest.json              # Global index
├── memory/
│   ├── MEMORY.md              # Memory index file
│   ├── user_student.md        # Memory files (Claude Code frontmatter format)
│   └── feedback_*.md
└── skills/
    └── brainstorming/         # Skill directory
        └── SKILL.md           # Skill definition
        └── *.md               # Auxiliary files
```

### manifest.json Schema

```json
{
  "version": 1,
  "updatedAt": "2026-05-09T12:00:00Z",
  "memory": [
    {
      "id": "user_student",
      "name": "Student Info",
      "description": "温州大学23网工1班",
      "type": "user",
      "path": "memory/user_student.md"
    }
  ],
  "skills": [
    {
      "id": "brainstorming",
      "name": "Brainstorming",
      "description": "Turn ideas into designs",
      "path": "skills/brainstorming/"
    }
  ]
}
```

### Memory File Format

Each memory file uses Claude Code frontmatter:
```markdown
---
name: Student Info
description: 温州大学23网工1班，学号22211357125
type: user
---

温州大学23网工1班学生，学号22211357125，正在学习网络智能技术与应用课程。
```

### Tech Stack

- Vanilla HTML/CSS/JS (single `index.html` file)
- JSZip (CDN) for zip generation
- GitHub REST API v3 for file operations
- localStorage for token persistence

## UI Design

### Placement

New section added after the existing Experience section in the portfolio page. Navigation bar gets a "Sync" link.

### Layout

```
┌─────────────────────────────────────────────────┐
│  Claude Sync                                    │
│  Manage your Claude Code memory & skills        │
│                                                 │
│  ┌─────────────────────────────────────────┐    │
│  │ Settings                                │    │
│  │ GitHub Token: [••••••••] [Save]         │    │
│  │ Repo: wlopyst/wlopyst.github.io         │    │
│  └─────────────────────────────────────────┘    │
│                                                 │
│  ┌── Memory ────────────┐ ┌── Skills ──────────┐│
│  │ [+ Upload] [Download] │ │ [+ Upload]         ││
│  │                       │ │                    ││
│  │ user_student.md       │ │ brainstorming/     ││
│  │   "Student Info"      │ │   "Turn ideas..."  ││
│  │   [Edit] [Delete]     │ │   [Edit] [Delete]  ││
│  └───────────────────────┘ └────────────────────┘│
│                                                 │
│  [Download All as ZIP]                          │
│                                                 │
│  ┌─────────────────────────────────────────┐    │
│  │ Editor (shown when editing)             │    │
│  │ ┌─────────────────────────────────────┐ │    │
│  │ │ (markdown content)                  │ │    │
│  │ └─────────────────────────────────────┘ │    │
│  │ [Cancel]  [Save & Push to GitHub]       │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

## Implementation Details

### GitHub API Interaction Layer (`ClaudeSync` class)

Methods:
- `init(token, owner, repo)` — initialize with credentials
- `loadManifest()` — GET `claude-sync/manifest.json`, parse and return
- `uploadFile(path, content, message)` — PUT create/update file
- `deleteFile(path, sha, message)` — DELETE file (requires SHA)
- `getFileContent(path)` — GET raw file content
- `updateManifest(manifest)` — PUT updated manifest.json

Authentication: `Authorization: token <PAT>` header. Token stored in `localStorage` key `claude-sync-token`.

### File Upload

1. `<input type="file" multiple>` for selecting files
2. Detect file type by path/name pattern:
   - `memory/*.md` → memory files
   - `skills/*/SKILL.md` → skill files
3. Parse Claude Code frontmatter (if present) to extract name/type/description
4. Upload via GitHub API PUT
5. Update manifest.json

### Online Editor

1. Click "Edit" on a file → fetch content via GitHub API
2. Display in `<textarea>` with monospace font
3. "Save & Push" → PUT updated content to GitHub
4. Auto-update manifest if metadata changed

### Package Download

1. Fetch manifest to get all file paths
2. Fetch each file's content via GitHub API
3. Generate zip with JSZip:
   ```
   claude-sync-export-YYYYMMDD/
   ├── memory/          → ~/.claude/projects/xxx/memory/
   ├── skills/          → ~/.claude/plugins/
   └── SETUP.md         → adaptation guide
   ```
4. Trigger browser download

### Security

- GitHub PAT stored only in browser localStorage, never in repo code
- Token input field uses `type="password"`
- "Clear Token" button to remove stored token
- No token transmitted to any third-party service

## New Device Adaptation Flow

1. Visit `wlopyst.github.io` → scroll to Claude Sync section
2. Click "Download All as ZIP"
3. Extract zip
4. Copy `memory/` files to `~/.claude/projects/<project>/memory/`
5. Copy `skills/` files to `~/.claude/plugins/`
6. Launch Claude Code — memory and skills auto-loaded

## Styling

Follow existing portfolio design tokens:
- Glass-morphism cards (`--glass-bg`, `--glass-border`)
- Gradient accents (`--gradient-start`, `--gradient-end`)
- Inter + JetBrains Mono fonts
- Scroll-reveal animations
- Responsive layout (stacked on mobile)

## Dependencies

- JSZip 3.x (CDN: `https://cdn.jsdelivr.net/npm/jszip@3/dist/jszip.min.js`)
- No other external dependencies
