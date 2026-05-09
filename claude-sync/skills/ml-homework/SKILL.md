---
name: ml-homework
description: 完成机器学习/数据科学课程作业 - 从docx模板提取需求、生成Python代码运行、将结果和图表写回docx。当用户提到"作业"、"机器学习作业"、"sklearn作业"、"分类作业"、"数据集分析"时使用。
---

# ML 课程作业自动化

## 这个 Skill 做什么

端到端完成机器学习/数据科学课程的书面作业：

1. **读取** docx 作业模板，提取需求和评分标准
2. **生成** Python 脚本完成数据分析、模型训练、可视化
3. **运行** 脚本，收集输出结果和图表
4. **写入** 新 docx 文件，包含代码、运行结果、图表和分析文字

## 何时使用

**合适的场景**：
- 课程作业要求用 sklearn 做分类/回归/聚类
- 需要在 docx 模板中填写代码、结果和图表
- 需要对比多种机器学习方法
- 需要数据集特征分析和可视化

**不合适的场景**：
- 深度学习项目（框架不同）
- 纯理论作业（不需要代码）
- 数据量极大的工程项目

## 工作流

### Step 1 · 需求提取（动手前必做）

用 python-docx 读取作业模板，提取所有要求：

```python
from docx import Document
doc = Document('作业模板.docx')
for p in doc.paragraphs:
    print(p.text)
for t in doc.tables:
    for row in t.rows:
        print(' | '.join(cell.text for cell in row.cells))
```

**关键动作**：
- 列出每项要求及其分值
- 确认需要的数据集（sklearn内置 或 自定义）
- 确认需要的方法数量和类型
- 确认需要的可视化类型（散点图、柱状图、热力图等）

### Step 2 · 环境检查

确认以下库已安装，缺失则安装：

```bash
pip install scikit-learn matplotlib numpy xgboost python-docx
```

检查中文字体支持（matplotlib 需要 SimHei 或 Microsoft YaHei）：

```python
plt.rcParams['font.sans-serif'] = ['SimHei', 'Microsoft YaHei', 'DejaVu Sans']
plt.rcParams['axes.unicode_minus'] = False
```

### Step 3 · 生成分析脚本

按作业要求结构组织 Python 脚本，每个要求对应一个代码块：

#### 3.1 数据集分析（通常20分）

```python
# 加载数据
from sklearn.datasets import load_iris  # 或其他数据集
data = load_iris()
X, y = data.data, data.target

# 基本统计
print(f"样本数: {X.shape[0]}, 特征数: {X.shape[1]}")
print(f"特征名: {data.feature_names}")
print(f"类别: {list(data.target_names)}")
print(f"各类别样本数: {np.bincount(y)}")

# 可视化：散点图矩阵、箱线图、相关性热力图
```

**输出要求**：
- 特征统计表格（均值、标准差、最值）
- 散点图矩阵（展示特征间关系和类别分布）
- 箱线图（各类别特征分布）
- 相关性热力图（特征间线性关系）
- （可选）特征重要性图（基于随机森林）

#### 3.2 模型实现（通常40分）

```python
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, accuracy_score, f1_score

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y)

# 对需要标准化的模型（LR、SVM、KNN）使用 StandardScaler
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)
```

**注意**：
- 逻辑回归 solver 选择：多分类问题不能用 `liblinear`，用 `lbfgs`/`newton-cg`/`sag`
- 朴素贝叶斯不需要标准化
- 随机森林、决策树不需要标准化
- 始终用 `cross_val_score` 评估泛化性能

#### 3.3 结果对比（通常20分）

```python
# 对比表格
# 柱状图：准确率、F1、交叉验证
# 各类别F1对比
# 混淆矩阵
```

**输出要求**：
- 结果对比表格（准确率、Macro F1、5折CV均值±标准差）
- 柱状图对比（至少2个维度：准确率、F1）
- 各类别F1分组柱状图
- 混淆矩阵热力图

#### 3.4 改进/调优（通常20分）

```python
# 网格搜索调参
from sklearn.model_selection import GridSearchCV

# 参数调优前后对比
# 额外方法对比
```

**输出要求**：
- 参数搜索范围和最优参数
- 调优前后对比柱状图
- （可选）额外方法（SVM、决策树等）对比

### Step 4 · 运行脚本

```bash
cd <作业目录> && python homework.py 2>&1
```

**检查清单**：
- [ ] 所有图片已生成到 output/ 目录
- [ ] 无运行错误
- [ ] 分类报告输出完整
- [ ] 交叉验证结果合理（不要出现明显过拟合）

### Step 5 · 写入 docx

用 python-docx 填充作业文件：

```python
from docx import Document
from docx.shared import Inches, Pt
from docx.oxml.ns import qn

doc = Document()
# 封面信息（课程名、班级、姓名、学号）
# 每个要求对应一个 Heading
# 代码块（等宽字体 + 灰色背景）
# 运行结果（代码块格式）
# 图片（Inches宽度，居中）
# 分析文字（宋体10.5pt）
```

**代码块样式**：
```python
def add_code_block(doc, code_text):
    p = doc.add_paragraph()
    run = p.add_run(code_text)
    run.font.name = 'Consolas'
    run.font.size = Pt(9)
    # 灰色背景
    shading = p._element.get_or_add_pPr()
    shd = shading.makeelement(qn('w:shd'), {
        qn('w:val'): 'clear', qn('w:color'): 'auto', qn('w:fill'): 'F5F5F5'
    })
    shading.append(shd)
```

**图片插入**：
```python
doc.add_picture('output/xxx.png', width=Inches(5.5))
doc.paragraphs[-1].alignment = WD_ALIGN_PARAGRAPH.CENTER
```

### Step 6 · 文件命名

按课程要求命名：`学号姓名作业N.docx`（如 `22211357125王涛作业1.docx`）

## 常见数据集选择

| 数据集 | 样本 | 特征 | 类别 | 适合 |
|--------|------|------|------|------|
| Iris（鸢尾花） | 150 | 4 | 3 | 入门分类 |
| Wine（葡萄酒） | 178 | 13 | 3 | 多特征分类 |
| Breast Cancer | 569 | 30 | 2 | 二分类 |
| Digits（手写数字） | 1797 | 64 | 10 | 多分类 |
| Wine Quality | 4898/6497 | 11 | 7/11 | 不平衡分类 |

## 常见坑

1. **liblinear 不支持多分类**：Iris/Wine 等3分类问题，逻辑回归用 `solver='lbfgs'`
2. **标准化范围**：LR、SVM、KNN 需要标准化；朴素贝叶斯、树模型不需要
3. **matplotlib 中文乱码**：必须设置 `font.sans-serif`
4. **图片不显示**：用 `matplotlib.use('Agg')` 后端，确保 `savefig` 在 `show` 之前
5. **docx 编码**：python-docx 内部处理 UTF-8，但 print 输出需 `sys.stdout.reconfigure(encoding='utf-8')`
6. **交叉验证 vs 测试集**：调参用 CV，最终评估用测试集，不要混用
7. **过拟合**：测试集准确率100%但CV很低 = 过拟合，需调整模型复杂度

## 文件结构约定

```
课后作业/
├── homework1_xxx.py      # 作业1分析脚本
├── homework2_xxx.py      # 作业2分析脚本
├── fill_homework.py      # docx填充脚本
├── output/               # 生成的图表
│   ├── 1_xxx.png
│   ├── 2_xxx.png
│   └── ...
├── 22211357125作业1.docx  # 最终提交文件
└── 22211357125作业2.docx
```

## 依赖

```
scikit-learn>=1.3
matplotlib>=3.7
numpy>=1.24
xgboost>=1.7       # 可选，集成方法作业需要
python-docx>=0.8   # 读写docx
```
