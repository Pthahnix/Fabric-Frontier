<!-- markdownlint-disable -->
# Rebuttal: Pattern Matching Constraint Solver

> Reviewer角色：跨领域审稿人（combinatorial optimization + textile manufacturing）
> 审阅日期：2026-03-13
> 审阅对象：ideas/12-pattern-matching-constraint-solver.md

## 主要反驳意见

### 1. 学术空白确实存在但验证基础薄弱——N=9合理，E=4是隐患

图案对花（stripe/plaid alignment）在学术文献中确实几乎没有被AI方法处理过——N=9的评分是合理的。但Evidence得分E=4反映了一个深层问题：**没有足够的先验工作意味着没有已知的evaluation protocol、没有benchmark数据集、没有baseline方法可以对比**。这不仅是"证据不足"，而是整个research infrastructure缺失。在这种情况下，即使方法本身正确，也难以被学术界接受——因为无法被reproducible地评估。

### 2. MIP solver的计算复杂度可能使方案不可行

Mixed Integer Programming在图案对花场景中面临的搜索空间是巨大的：对于一件有N个版型片、每片有M个可能旋转角度和P个可能平移位置的排版问题，变量空间为O(N×M×P)，约束数量与片对数量成二次增长。实际工业场景中N可达20-50片，M和P在连续化后需要足够细的离散化。Gradient Fields Packing（2023）的**diffusion-based relaxation**提供了更promising的路径——它在garment数据上已经超越了teacher policy，且能自然处理连续空间优化，避免了离散化带来的精度-效率权衡。

### 3. 可微松弛（differentiable relaxation）是更优方案

将图案对花formulate为可微优化问题有三个关键优势：

- **端到端训练**：可以与pattern generation联合优化，让生成模型直接输出易于对花的版型
- **梯度信息**：比MIP的branch-and-bound搜索更高效
- **Learning-Based 2D Packing**（2023）已证明learning-based方案在packing问题上有5-10%的improvement，且**domain transfer works**——从一般packing到garment-specific packing的迁移是可行的

MIP solver应定位为验证工具（检查最终方案是否满足对花约束），而非主要求解器。

### 4. 面料批次差异——被忽略的不确定性

实际工业生产中，面料的图案重复单元（repeat）存在**印刷误差**。标称repeat为10cm的条纹面料，实际可能在9.7-10.3cm之间波动，且同一卷面料不同位置的repeat可能不同。此外，面料在裁剪和缝制过程中会发生**拉伸和变形**，进一步改变图案的对齐。Idea将图案对花建模为确定性约束满足问题，完全忽略了这种manufacturing uncertainty。Textile IR（2026）报告的compound uncertainty约26%，虽然不是直接关于图案对花，但说明了服装制造中uncertainty的普遍性。

### 5. 实际对花的"可接受范围"是主观的

工业中的图案对花标准因产品层次不同而差异巨大：高级定制要求完美对花（tolerance < 1mm），商业成衣允许2-3mm偏差，快时尚可能完全不对花。Idea没有讨论如何parameterize这种quality-cost tradeoff——更严格的对花约束意味着更高的面料利用损失。需要一个Pareto前沿分析，展示对花精度vs面料利用率的权衡曲线。

### 6. 多层叠加对花的组合爆炸

许多服装不仅需要相邻版型片对花，还需要在多层叠加时对花（如外层面料与里布的格纹对齐）。Dress-1-to-3的CIPC multi-layer方法处理了多层布料的物理交互，但图案对花在多层场景下的约束空间会产生组合爆炸。Idea需要讨论对多层场景的scalability。

## 改进建议

1. **将MIP重新定位为验证层**而非主要求解器——使用Gradient Fields Packing的diffusion-based方法作为主要求解引擎，MIP作为feasibility checker
2. **引入uncertainty modeling**——将面料repeat的制造误差建模为constraint的tolerance band而非fixed values
3. **设计quality-cost Pareto分析**——展示对花精度vs面料利用率的tradeoff frontier
4. **建立benchmark数据集**——既然E=4，首要任务是创建可公开的对花评估数据集和evaluation protocol
5. **考虑与pattern generation的端到端集成**——让生成模型学会输出"对花友好"的版型排布
6. **讨论multi-layer对花的scalability**——至少在future work中承认这个问题

## 评分调整建议

| 维度 | 原分 | 建议 | 理由 |
|------|------|------|------|
| Novelty (N) | 9 | 9 | 确实是学术空白，不调整 |
| Feasibility (F) | 5 | 4 | MIP复杂度+缺少验证基础使可行性更低 |
| Impact (I) | 7 | 7 | 工业价值真实但受限于特定面料类型 |
| Confidence (C) | 7 | 6 | Manufacturing uncertainty未被考虑降低confidence |
| Evidence (E) | 4 | 4 | 不变，确实缺少先验工作 |
| Total | 32 | 30 | 值得探索但技术路径需要重新设计 |

## 补充参考文献

- **Gradient Fields Packing** (2023): Diffusion-based packing, surpasses teacher on garment data
- **Learning-Based 2D Packing** (2023): 5-10% improvement, domain transfer viable
- **Textile IR** (2026): Compound uncertainty ~26% in textile manufacturing
- **Dress-1-to-3**: CIPC multi-layer cloth interaction — relevant to multi-layer pattern matching
- **FALCON** (2026): Constraint satisfaction with 100% feasibility — potential verification layer
