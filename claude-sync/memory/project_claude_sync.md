---
name: Claude Sync Feature
description: wlopyst.github.io 个人作品集网站新增 Claude Sync 功能，管理 Claude Code 的 memory 和 skills 文件
type: project
originSessionId: 852bf0b4-20e1-41a2-b7cf-12fca4a0faea
---
wlopyst.github.io 个人作品集网站已实现 Claude Sync 功能（2026-05-09）。

**功能：** 通过网页界面管理 Claude Code 的 memory 文件和 skills 文件，支持上传、编辑、删除、打包下载。

**技术栈：** 纯前端单页应用（index.html），GitHub REST API v3 存储数据，JSZip 打包下载。

**仓库结构：**
- `claude-sync/manifest.json` — 全局索引
- `claude-sync/memory/*.md` — 记忆文件（Claude Code frontmatter 格式）
- `claude-sync/skills/*/` — 技能目录

**关键实现：**
- `ClaudeSync` 类封装 GitHub API（loadManifest, uploadFile, deleteFile, getFile, saveManifest）
- Token 存储在 localStorage，不写入仓库
- 下载 zip 包含 SETUP.md 适配指南，目录结构匹配 `~/.claude/projects/xxx/memory/` 和 `~/.claude/plugins/`

**Why:** 用户经常更换设备，需要快速将 Claude Code 的记忆和技能迁移到新环境。

**How to apply:** 后续修改此功能时，参考 `docs/superpowers/specs/2026-05-09-claude-sync-design.md` 设计文档和 `docs/superpowers/plans/2026-05-09-claude-sync.md` 实施计划。
