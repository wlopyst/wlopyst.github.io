---
name: ml-homework-workflow
description: ML作业的最佳工作流 - 先读模板提需求，再写Python脚本运行，最后用python-docx填充docx
type: feedback
originSessionId: 3d0651b0-4601-4058-a0db-7aa680dc0f34
---
完成ML课程作业的最佳工作流：先用python-docx读取模板提取需求 → 写完整Python脚本（分析+可视化+模型训练+评估）→ 运行生成图片到output/ → 再写一个fill脚本把代码、结果、图片写入新docx。

**Why:** 分开脚本和填充逻辑便于调试。如果脚本报错可以单独修复，不影响docx结构。图片生成和docx写入解耦，避免重复运行耗时的模型训练。

**How to apply:**
1. 分析脚本（homework1_xxx.py）负责数据加载、模型训练、图表生成，所有print输出应包含完整结果
2. 填充脚本（fill_homework.py）负责读取模板结构、插入代码/结果/图片/分析文字
3. 两个脚本共享同一个 output/ 目录存放图片
4. 运行顺序：分析脚本 → 填充脚本 → 检查docx文件大小（含图片应>200KB）
