---
name: seminar-report
description: 完成文献研讨报告 - 批量读取PDF论文、提取核心内容、按模板结构生成docx报告。当用户提到"研讨报告"、"文献阅读"、"论文研讨"、"工程问题研讨"时使用。
---

# 文献研讨报告自动化

## 这个 Skill 做什么

端到端完成课程文献研讨报告：

1. **批量读取** PDF论文，提取摘要、方法、结论等核心内容
2. **分析对比** 多篇文献的方法异同，形成系统性梳理
3. **按模板结构** 生成docx报告，含对比表格、数据表格、参考文献

## 何时使用

**合适的场景**：
- 课程要求阅读多篇论文（通常≥8篇）并撰写研讨报告
- 需要对比分析不同方法的优劣
- 报告有固定模板结构（问题分析→方法比较→数据总结→研讨总结）
- 需要生成规范的参考文献列表

**不合适的场景**：
- 单篇论文的精读报告
- 需要实验复现的作业（用 ml-homework skill）
- 纯综述类论文（不需要方法对比）

## 工作流

### Step 1 · 需求提取（动手前必做）

读取报告模板，确认4项要求和评分标准：

```python
from docx import Document
doc = Document('研讨报告模板.docx')
for p in doc.paragraphs:
    print(p.text)
```

**关键动作**：
- 确认报告结构（通常4部分：问题分析→方法比较→数据总结→总结）
- 确认评分标准（文献数量要求、复现/仿真要求）
- 确认提交截止日期和命名规范
- 确认可选主题数量，让用户选择

### Step 2 · 批量读取PDF论文

安装依赖并提取所有PDF的核心内容：

```bash
pip install PyPDF2 python-docx
```

```python
from PyPDF2 import PdfReader
import os

pdf_dir = '主题目录/'
for f in sorted(os.listdir(pdf_dir)):
    if f.endswith('.pdf'):
        reader = PdfReader(os.path.join(pdf_dir, f))
        text = ''
        for page in reader.pages[:6]:  # 前6页通常覆盖摘要+方法
            t = page.extract_text()
            if t:
                text += t + '\n'
        print(f'===== {f} =====')
        print(text[:3000])  # 3000字符足够抓核心
```

**注意**：
- 扫描版PDF（图片型）PyPDF2无法提取文本，需跳过或用OCR
- 演示文稿型PDF（如会议slides）提取效果差，需从标题和图片推断内容
- 每篇论文记录：标题、作者、年份、核心方法、主要结论、关键数据

### Step 3 · 组织报告结构

按模板的4部分结构组织内容：

#### 第一部分：工程问题需求分析（约1500字）

```
1.1 问题背景
    - 领域现状和挑战（3-5个核心挑战）
    - 数据规模、复杂性、标注匮乏等

1.2 工程问题定义
    - 核心问题一句话定义
    - 3-4个子问题（从文献中提炼）

1.3 研讨范围与目标
    - 覆盖哪几个研究方向
    - 基于多少篇文献
```

#### 第二部分：求解方法分析与比较（约2500字）

```
2.1 方向一的方法（如：自动化算法设计）
    - 方法A：核心思想、LLM角色、创新点
    - 方法B：...
    - 方法C：...

2.2 方向二的方法（如：网络部署优化）
    - ...

2.3 方向三的方法（如：智能运维）
    - ...

2.4 方法对比分析（**必须有表格**）
    表格列：方法名 | 核心思想 | LLM角色 | 优势 | 局限性
```

**方法描述模板**：
```
[作者]等人[N]提出了[方法名]，核心思想是[一句话概括]。
该方法将LLM作为[角色]，通过[机制]实现了[目标]。
在[问题]上的实验表明，[关键结果]。
```

#### 第三部分：数据分析说明（约1500字）

```
3.1 方向一的性能数据
    - 方法A的具体指标（提升百分比、准确率等）
    - 方法B的具体指标

3.2 方向二的性能数据
    - ...

3.3 方向三的性能数据
    - ...

3.4 方法间的性能对比总结（**必须有表格**）
    表格列：评估维度 | 方法类A | 方法类B | 方法类C
```

#### 第四部分：研讨总结（约1000字）

```
4.1 关键发现（3-5条，每条一句话标题+展开说明）

4.2 现有挑战与不足（3-5条）

4.3 未来发展趋势（3-4条）

4.4 个人思考（1段，结合专业背景）
```

#### 参考文献

格式：`[序号] 作者. 标题[J/C/R]. 期刊/会议, 年份.`

- [J] = 期刊论文
- [C] = 会议论文
- [R] = 报告/技术报告
- 数量要求：≥8篇（80-100分档），4-7篇（60-79分档）

### Step 4 · 生成docx

```python
from docx import Document
from docx.shared import Pt, RGBColor
from docx.oxml.ns import qn

def set_font(run, name='宋体', size=10.5, bold=False):
    run.font.name = name
    run.font.size = Pt(size)
    run.font.bold = bold
    run._element.rPr.rFonts.set(qn('w:eastAsia'), name)

def add_para(doc, text, bold=False, indent=True):
    p = doc.add_paragraph()
    if indent:
        p.paragraph_format.first_line_indent = Pt(21)
    run = p.add_run(text)
    set_font(run, bold=bold)
```

**docx结构检查清单**：
- [ ] 封面含课程名、班级、姓名、学号
- [ ] 4个一级标题（Heading 1）
- [ ] 方法对比表格（≥7行×5列）
- [ ] 数据对比表格（≥4行×4列）
- [ ] 参考文献≥8篇
- [ ] 总字数6000-8000
- [ ] 文件名格式：`学号-研讨报告.docx`

### Step 5 · 验证

```python
from docx import Document
doc = Document('报告.docx')
print(f'段落数: {len(doc.paragraphs)}')
print(f'表格数: {len(doc.tables)}')
total = sum(len(p.text) for p in doc.paragraphs)
print(f'总字数: {total}')

# 检查文献数量
ref_count = sum(1 for p in doc.paragraphs if p.text.strip().startswith('['))
print(f'参考文献数: {ref_count}')
```

## PDF提取常见问题

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 提取为空 | 扫描版PDF（图片） | 跳过，从标题推断内容 |
| 乱码 | 编码问题 | 尝试不同encoding或跳过 |
| 只有部分内容 | PDF结构复杂 | 增加读取页数到10页 |
| 公式/图表丢失 | PyPDF2不支持 | 忽略，关注文字描述 |

## 常见坑

1. **文献数量不够**：每篇论文必须在报告中明确引用（[N]标注），否则不算有效文献
2. **方法描述太泛**：每篇论文必须写出具体的方法名称和核心创新点，不能只说"提出了XX方法"
3. **缺少对比表格**：方法对比和数据对比必须有表格，纯文字描述会扣分
4. **参考文献格式**：必须标注类型[J]期刊/[C]会议/[R]报告，格式不规范可能不被认可
5. **个人思考太空**：要结合自身专业背景（如网络工程），不能泛泛而谈
6. **数据没有来源**：所有性能数据必须标注来自哪篇文献[N]

## 文件结构

```
研讨材料/
├── 主题名/              # 原始PDF论文
│   ├── paper1.pdf
│   ├── paper2.pdf
│   └── ...
├── generate_report.py   # 报告生成脚本
└── 学号-研讨报告.docx   # 最终提交文件
```

## 依赖

```
PyPDF2>=3.0       # PDF文本提取
python-docx>=0.8  # 读写docx
```
