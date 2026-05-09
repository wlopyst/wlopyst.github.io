---
name: course-final-design
description: 完成课程期末设计报告 - 数据集分析、多方法实现、结果对比可视化、按模板生成docx含代码附录。当用户提到"期末设计"、"课程设计"、"大作业"、"流量分类"、"网络流量识别"时使用。
---

# 课程期末设计自动化

## 这个 Skill 做什么

端到端完成课程期末设计报告：

1. **分析** 数据集特征，生成可视化图表
2. **实现** 多种机器学习/深度学习方法
3. **对比** 方法性能，生成10+张图表
4. **按模板** 生成完整docx报告（需求分析→方法设计→方法实现→结果分析→总结+代码附录）

## 何时使用

**合适的场景**：
- 课程期末设计/课程设计/大作业
- 需要实现多种方法并对比
- 需要大量可视化图表
- 报告有固定模板（通常5-6部分）
- 需要代码附录

**不合适的场景**：
- 普通课后作业（用 ml-homework skill）
- 文献研讨报告（用 seminar-report skill）
- 毕业设计（规模更大，需要更完整的流程）

## 工作流

### Step 1 · 需求提取（动手前必做）

读取模板确认5项要求和评分标准：

```python
from docx import Document
doc = Document('期末设计模板.docx')
for p in doc.paragraphs:
    if p.text.strip():
        print(f'[{p.style.name}] {p.text}')
```

**关键动作**：
- 确认每个部分的分值分配
- 确认需要的方法数量（通常≥2种）
- 确认参考文献数量要求（通常≥3篇）
- 确认是否需要社会影响/伦理讨论
- 确认提交截止日期和命名规范

### Step 2 · 数据集准备

**优先使用真实数据集**，下载失败则生成模拟数据：

#### 真实数据集来源

| 数据集 | 样本数 | 特征数 | 类别 | 格式 |
|--------|--------|--------|------|------|
| UNSW-NB15 | 训练82332+测试175341 | 39数值 | 10 | CSV(tab分隔) |
| CICIDS2017 | 283万 | 78 | 15 | CSV |
| KDD Cup 1999 | 490万 | 41 | 5 | CSV |
| NSL-KDD | 14.8万 | 41 | 5 | CSV |

#### UNSW-NB15 加载代码（已验证可用）

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import LabelEncoder

DATA_DIR = 'C:/Users/lanty/Desktop/数据集/UNSW_NB15_CSV'
df_train = pd.read_csv(f'{DATA_DIR}/UNSW_NB15_training-set.csv', sep='\t')
df_test = pd.read_csv(f'{DATA_DIR}/UNSW_NB15_testing-set.csv', sep='\t')

# 数值特征（排除id和label）
numeric_cols = df_train.select_dtypes(include=[np.number]).columns.tolist()
feature_cols = [c for c in numeric_cols if c not in ['id', 'label']]

# 缺失值和inf处理
df_train = df_train.dropna(subset=feature_cols)
df_test = df_test.dropna(subset=feature_cols)
df_train[feature_cols] = df_train[feature_cols].replace([np.inf, -np.inf], np.nan).fillna(0)
df_test[feature_cols] = df_test[feature_cols].replace([np.inf, -np.inf], np.nan).fillna(0)

X_train = df_train[feature_cols].values
X_test = df_test[feature_cols].values

# 标签编码
le = LabelEncoder()
le.fit(np.concatenate([df_train['attack_cat'].values, df_test['attack_cat'].values]))
y_train = le.transform(df_train['attack_cat'].values)
y_test = le.transform(df_test['attack_cat'].values)
class_names = le.classes_  # 10个类别
```

**UNSW-NB15 类别分布**（训练集）：
- Normal: 37000, Generic: 18871, Exploits: 11132, Fuzzers: 6062
- DoS: 4089, Reconnaissance: 3496, Analysis: 677, Backdoor: 583
- Shellcode: 378, Worms: 44
- **注意**：存在严重类别不平衡，Worms仅44样本，Macro F1会显著低于准确率

#### 模拟数据集生成策略（备选）

当真实数据集不可用时，参照目标数据集的特征体系生成模拟数据：

```python
def generate_class_data(cls_idx, n_samples, n_features):
    """为每个类别生成不同分布的特征数据"""
    base = np.random.randn(n_samples, n_features)
    # 根据类别特点调整各特征的分布参数
    # Normal: 适中的包数/字节数，较低速率
    # DoS: 极高速率，大量小包，短持续时间
    # Exploits: 异常大载荷，中等速率
    # ...
    return base
```

**关键原则**：
- 特征名称和含义应与真实数据集一致
- 各类别的分布差异应模拟真实攻击行为
- 样本数建议每类1000-3000，总计5000-15000
- 类别数建议3-5个

### Step 3 · 方法实现

**至少实现2种方法，建议6种以获得更好的对比效果**：

#### 常用方法组合

```python
methods = {
    '随机森林 (RF)': RandomForestClassifier(n_estimators=200, max_depth=20, min_samples_split=5, random_state=42, n_jobs=-1),
    '梯度提升 (GBDT)': GradientBoostingClassifier(n_estimators=200, learning_rate=0.1, max_depth=5, random_state=42),
    '极端随机树 (ET)': ExtraTreesClassifier(n_estimators=200, max_depth=20, random_state=42, n_jobs=-1),
    'AdaBoost': AdaBoostClassifier(n_estimators=100, learning_rate=0.5, random_state=42),
    'K近邻 (KNN)': KNeighborsClassifier(n_neighbors=5, n_jobs=-1),
    '决策树 (DT)': DecisionTreeClassifier(max_depth=20, random_state=42),
}
```

#### 数据标准化

```python
# 标准化（KNN需要，树模型不需要）
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 训练时：KNN用X_train_scaled，其他用X_train
# 预测时：KNN用X_test_scaled，其他用X_test
```

#### 真实数据集参考性能（UNSW-NB15）

| 方法 | 准确率 | Macro F1 | Weighted F1 |
|------|--------|----------|-------------|
| 梯度提升 (GBDT) | 0.7584 | 0.4871 | 0.7290 |
| 随机森林 (RF) | 0.7505 | 0.4613 | 0.7179 |
| 决策树 (DT) | 0.7413 | 0.4722 | 0.7126 |
| 极端随机树 (ET) | 0.7165 | 0.3777 | 0.6720 |
| K近邻 (KNN) | 0.7060 | 0.3748 | 0.6725 |
| AdaBoost | 0.6638 | 0.3005 | 0.6111 |

**注意**：真实数据Macro F1显著低于准确率，原因是类别不平衡（Worms仅44样本）。

#### 评估指标

```python
acc = accuracy_score(y_test, y_pred)
f1_macro = f1_score(y_test, y_pred, average='macro')
f1_weighted = f1_score(y_test, y_pred, average='weighted')
f1_per_class = f1_score(y_test, y_pred, average=None)
cv_scores = cross_val_score(model, X, y, cv=5)
```

### Step 4 · 可视化图表（至少8张）

**必做图表**：

| 序号 | 图表 | 用途 | 代码 |
|------|------|------|------|
| 1 | 特征分布图 | 展示数据集特点 | `plt.hist()` 多类别叠加 |
| 2 | PCA降维可视化 | 展示类别可分性 | `PCA(n_components=2)` + `scatter` |
| 3 | 类别分布 | 展示样本均衡性 | `plt.pie()` |
| 4 | 综合对比柱状图 | 方法性能对比 | `plt.bar()` 3个指标并排 |
| 5 | 各类别F1对比 | 各方法在各类别表现 | `plt.bar()` 分组柱状图 |
| 6 | F1热力图 | 方法×类别交叉分析 | `plt.imshow()` |
| 7 | 混淆矩阵 | 最优方法详细分析 | `confusion_matrix()` + 热力图 |
| 8 | ROC曲线 | 模型区分能力 | `roc_curve()` + `auc()` |

**选做图表**：
- 特征相关性热力图
- 特征重要性排序
- 训练时间对比
- PR曲线
- 学习曲线

### Step 5 · 生成docx报告

#### 报告结构（按模板）

```
封面（项目名称、姓名、学号、班级、实验室、学期）
评分表
0 背景描述（简要保留模板内容）
1 需求分析（10分）
  1.1 数据集选择与分析
  1.2 数据集构建（如用模拟数据）
  1.3 设计目标
2 方法设计（30分）
  2.1 可行方法分析
  2.2 本设计采用的方法（含表格）
  2.3 参考文献（≥3篇，正文中用[N]标注）
3 方法实现（30分）
  3.1 数据准备（代码块）
  3.2 模型训练与评估（代码块）
4 结果分析（20分）
  4.1 整体性能对比（表格+图表）
  4.2 各类别识别效果分析
  4.3 混淆矩阵分析
  4.4 ROC曲线分析
  4.5 训练效率分析（可选）
  4.6 PCA降维可视化（可选）
5 总结（10分）
  5.1 设计总结
  5.2 智能方法对社会的影响（正面+负面）
  5.3 个人学习体会
代码附录（完整Python脚本）
```

#### 分值分配与写作重点

| 部分 | 分值 | 写作重点 |
|------|------|---------|
| 需求分析 | 10分 | 数据集特点要详细，目标要明确 |
| 方法设计 | 30分 | 方法分析要深入，文献引用要规范 |
| 方法实现 | 30分 | 代码要完整可运行，关键注释 |
| 结果分析 | 20分 | 图表要多维，文字分析要到位 |
| 总结 | 10分 | 必须含社会/法律/安全/伦理影响 |

#### 参考文献格式

```
[1] 作者. 标题[J/C]. 期刊/会议, 年份: 页码.
[2] 作者. 标题[J/C]. 期刊/会议, 年份: 页码.
```

- [J] = 期刊论文, [C] = 会议论文
- 数量要求：≥3篇（方法设计部分末尾列出）
- 正文中用[N]标注引用位置

### Step 6 · 验证

```python
from docx import Document
doc = Document('报告.docx')
print(f'段落数: {len(doc.paragraphs)}')
print(f'表格数: {len(doc.tables)}')
print(f'图片数: {len(doc.inline_shapes)}')
total = sum(len(p.text) for p in doc.paragraphs)
print(f'总字数: {total}')

# 检查参考文献
ref_count = sum(1 for p in doc.paragraphs if p.text.strip().startswith('['))
print(f'参考文献数: {ref_count}')

# 检查图片数
print(f'图片数: {len(doc.inline_shapes)} (建议≥8)')
```

**检查清单**：
- [ ] 封面含项目名称、姓名、学号、班级
- [ ] 评分表完整
- [ ] 5个主要部分（需求分析/方法设计/方法实现/结果分析/总结）
- [ ] 参考文献≥3篇，正文中有[N]标注
- [ ] 图表≥8张
- [ ] 代码附录包含完整Python脚本
- [ ] 总字数15000-25000
- [ ] 文件名格式：`学号-期末设计.docx`

## 常见坑

1. **数据集下载失败**：用模拟数据时务必参照真实数据集的特征体系，不能随便造数据
2. **UNSW-NB15是tab分隔**：读取时必须加 `sep='\t'`，否则所有数据会挤在一列
3. **类别不平衡**：UNSW-NB15的Worms仅44样本，Macro F1会显著低于准确率（约30%差距），这是正常的，需要在报告中分析原因
4. **inf/NaN处理**：真实数据中存在inf值，需用 `replace([np.inf, -np.inf], np.nan).fillna(0)` 处理
5. **方法太单一**：只实现1种方法会严重扣分，至少2种，建议6种
6. **图表不够**：少于6张图会扣分，建议8-12张覆盖多维度
7. **参考文献不够**：少于3篇不达标，且必须在正文中用[N]标注
8. **总结太浅**：必须讨论社会/法律/安全/伦理影响，不能只写技术总结
9. **代码不可运行**：附录的代码必须能独立运行，不能有未定义的变量
10. **混淆矩阵/ROC缺失**：这是最能体现分析深度的图表，必须包含
11. **真实数据运行慢**：UNSW-NB15有82332训练样本，GBDT等串行方法需要几分钟，设置足够timeout

## 与其他 Skill 的关系

| Skill | 用途 | 与本Skill的区别 |
|-------|------|----------------|
| ml-homework | 普通课后作业 | 规模小，2-3种方法，4-6张图 |
| seminar-report | 文献研讨报告 | 不需要代码实现，侧重文献分析 |
| course-final-design | 期末设计 | 最综合，6种方法，10+张图，含代码附录 |

## 文件结构

```
期末设计/
├── output/                     # 生成的图表
│   ├── 1_特征分布.png
│   ├── 2_PCA可视化.png
│   └── ...（8-12张图）
├── traffic_classification.py   # 完整分析脚本
├── generate_report.py          # 报告生成脚本
└── 学号-期末设计.docx          # 最终提交文件
```

## 依赖

```
scikit-learn>=1.3    # ML模型
matplotlib>=3.7      # 可视化
numpy>=1.24          # 数值计算
python-docx>=0.8     # 读写docx
```
