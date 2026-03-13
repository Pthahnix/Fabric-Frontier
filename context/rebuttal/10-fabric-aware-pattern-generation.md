<!-- markdownlint-disable -->
# Rebuttal: Fabric-Aware Pattern Generation

> Reviewer角色：跨领域审稿人（computational physics + generative models）
> 审阅日期：2026-03-13
> 审阅对象：ideas/10-fabric-aware-pattern-generation.md

## 主要反驳意见

### 1. Constraint injection优于soft conditioning——FALCON和SCIGEN的启示

该Idea提出通过AdaLN或cross-attention将面料属性注入生成模型。然而FALCON（2026）展示了一种更可靠的路径：**grammar-constrained decoding + repair + sampling实现了100% feasibility guarantee**。面料约束（如最小缝份宽度受面料厚度限制、弹性面料的回缩补偿）本质上是hard constraints——soft conditioning（AdaLN）无法保证这些约束被满足。SCIGEN（MIT, 2025）进一步证明了在生成过程中注入物理约束（而非post-hoc filtering）的有效性。面料属性应作为generation-time constraint参与decoding，而非仅作为condition影响分布。

### 2. DiffAvatar已恢复材料参数但未反馈到版型——这是切入点也是警告

DiffAvatar已经能够从视频恢复材料参数（弹性模量、弯曲刚度等），但这些参数仅用于simulation而不反馈到pattern generation。这确实是该Idea的核心切入点——将recovered material parameters闭环到版型生成。但这也揭示一个问题：**如果DiffAvatar团队没有做这个闭环，可能有技术原因**。材料参数到版型调整的映射是高度非线性的——同样的弹性模量变化，对不同garment部位的影响完全不同。Idea需要更明确地分析为什么现有工作没有实现这个闭环，以及自己的方案如何克服这些障碍。

### 3. VLM无法可靠处理定量物理属性——QuantiPhy的严厉警告

QuantiPhy（2025）的发现对任何涉及物理量的AI系统都是重要警告：**最佳VLM仅53.1% MRA vs人类55.6%**，且Chain-of-Thought在19/21个模型上反而降低性能。更关键的是，MRA在counterfactual priors上下降80%——这意味着**VLM并非真正理解物理量，而是在memorize常见数值**。面料属性（如经向弹性系数2.3 N/m、纬向弯曲刚度0.8 mN·m）是精确的定量输入，绝不能依赖VLM的"理解"。该Idea必须确保面料属性通过结构化数值通道（而非language embedding）输入模型。

### 4. 缺少ablation计划——哪些面料属性真正重要？

面料属性维度众多：重量（g/m²）、厚度、经纬向弹性模量、弯曲刚度、剪切刚度、摩擦系数、悬垂系数、回复率等。Idea提出将这些作为"first-class input"，但没有讨论**哪些属性对版型生成影响最大**。工业实践中，版型师通常只关注3-4个关键属性（手感/厚度/弹性/悬垂性）。全部属性输入可能引入噪声。需要一个系统的ablation study计划来确定最优属性子集。

### 5. 面料属性的获取本身是瓶颈

即使模型能完美利用面料属性，**这些属性从哪里来？** 工业面料数据库（如FABRIC-DB）覆盖率有限，实时测量需要专用设备（KES-F、FAST系统），手感评估高度主观。Idea假设面料属性是可用的结构化输入，但在实际workflow中，这些数据的获取成本和准确性是主要瓶颈。

### 6. DeepPHY的警告：描述性理解≠预测性控制

DeepPHY（2025）发现"even state-of-the-art VLMs struggle to translate descriptive physical knowledge into precise, predictive control"，且大多数open-source模型无法超越random baseline。这意味着即使模型"知道"面料属性会影响版型，也不代表它能正确预测应该如何调整版型。从"面料弹性增大"到"袖窿深度减少2mm"的推理链条需要精确的物理建模，不能依赖statistical pattern matching。

## 改进建议

1. **将架构从soft conditioning转向constraint injection**——参考FALCON的grammar-constrained decoding，将关键面料约束（最小缝份、回缩补偿）编码为hard constraints
2. **建立在DiffAvatar基础上**——明确引用DiffAvatar的材料参数恢复，并解释如何将其闭环到pattern generation
3. **设计结构化面料属性输入通道**——避免通过language embedding处理数值，使用dedicated numerical encoder
4. **制定ablation study计划**——确定最小有效属性子集（建议从3-4个开始：弹性/弯曲刚度/厚度/重量）
5. **讨论面料数据获取策略**——如何在没有实验室测量设备的情况下估计面料属性（如从面料图片/视频估计）

## 评分调整建议

| 维度 | 原分 | 建议 | 理由 |
|------|------|------|------|
| Novelty (N) | 7 | 7 | 面料感知生成确实是gap |
| Feasibility (F) | 6 | 5 | 面料数据获取瓶颈+constraint injection比soft conditioning更难 |
| Impact (I) | 9 | 8 | 影响取决于面料数据的可用性 |
| Confidence (C) | 8 | 7 | QuantiPhy/DeepPHY对定量物理推理的质疑降低了confidence |
| Evidence (E) | 7 | 7 | 不变 |
| Total | 37 | 34 | 方向正确但技术路径需要重新设计 |

## 补充参考文献

- **FALCON** (2026): Grammar-constrained decoding, 100% feasibility guarantee
- **SCIGEN** (MIT, 2025): Physics constraint injection during generation
- **QuantiPhy** (2025): VLM quantitative reasoning failure, CoT hurts 19/21 models
- **DeepPHY** (2025): Descriptive vs predictive gap in VLM physical understanding
- **DiffAvatar**: Material parameter recovery from video — potential upstream for fabric properties
- **DiffCloth**: Differentiable cloth simulation with system identification
