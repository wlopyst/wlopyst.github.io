---
name: final-design-workflow
description: 期末设计最佳流程 - 优先用真实UNSW-NB15数据集→6种方法实现→10+张图表→5部分+代码附录生成docx
type: feedback
originSessionId: 3d0651b0-4601-4058-a0db-7aa680dc0f34
---
完成期末设计报告的最佳流程：优先使用真实数据集（UNSW-NB15），数据量大时需耐心等待训练完成 → 实现6种分类方法 → 生成10+张可视化图表 → 按模板5部分结构写入docx，代码附在末尾。

**Why:** 真实数据集的结果更有说服力（模拟数据容易过拟合到100%准确率）。UNSW-NB15有类别不平衡问题（Worms仅44样本），Macro F1会显著低于准确率，这是正常的，需要在报告中分析原因。

**How to apply:**
1. 数据集：优先用真实UNSW-NB15（用户桌面数据集中有），tab分隔，39个数值特征，10类别
2. 预处理：dropna + replace inf为0，LabelEncoder编码attack_cat
3. 方法数量：6种（RF/GBDT/ET/AdaBoost/KNN/DT），KNN需要StandardScaler
4. 图表至少8张，命名用数字前缀（1_xxx.png~11_xxx.png）便于排序
5. 参考文献≥3篇，正文中用[N]标注
6. 总结必须含社会/法律/安全/伦理影响
7. 代码附录引用`traffic_classification_real.py`（非模拟版）
8. 真实数据运行时间较长（5-10分钟），需要足够timeout
