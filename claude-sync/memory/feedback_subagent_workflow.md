---
name: Subagent Workflow for Single-File Projects
description: 对于单文件项目（如 index.html），批量派发子代理效率更高，但需注意子代理可能跑偏
type: feedback
originSessionId: 852bf0b4-20e1-41a2-b7cf-12fca4a0faea
---
对于单文件项目的多任务实施，将相关任务批量派发给单个子代理比逐个派发更高效。

**Why:** 在 wlopyst.github.io 项目中，10 个任务全部修改同一个 index.html。逐个派发子代理（每个任务 3 个子代理：实现+spec review+quality review）开销过大。将 Tasks 5-9（5 个相关 JS 逻辑任务）合并为一次派发，效率显著提高。

**How to apply:**
- 单文件项目：将同一模块的多个小任务合并为一次子代理派发
- 多文件项目：仍按任务逐个派发，避免冲突
- 子代理可能跑偏（如 Task 4 的子代理生成了网络安全报告而非写代码），需及时 re-dispatch 并给出更明确的指令
- 简单任务（创建文件、添加 CSS）可以跳过完整 review 流程，直接验证 commit
