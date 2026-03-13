<!-- markdownlint-disable -->
# Rebuttal: Neural Pattern Grading

> Reviewer角色：跨领域审稿人（geometric deep learning + apparel engineering）
> 审阅日期：2026-03-13
> 审阅对象：ideas/11-ai-neural-pattern-grading.md

## 主要反驳意见

### 1. "没有任何AI方法"的前提需要修正——Inverse Garment和Dress Anyone已实现自动推板

这是最关键的反驳。Inverse Garment（2024）Section 3.3明确描述了**Pattern Linear Grading**方法：通过control point relocation实现自动化推板，本质上是在参数化版型表示上的线性插值。更重要的是，Dress Anyone（2024）实现了**automatic refitting to arbitrary body shapes**，且在5-30分钟内完成，并经过**真实缝制验证**。这两项工作直接否定了"推板领域没有AI方法"的claim。该Idea的GNN方案需要证明自己优于这些differentiable simulation-based的推板方案——而不是声称自己是第一个。

### 2. 需要与differentiable simulation方案进行系统性比较

Inverse Garment的方法基于differentiable rendering和optimization，Dress Anyone基于differentiable simulation——两者都是"learn to grade"的替代范式。GNN方案的核心主张是**学习工业推板规则**（discrete size jumps、grain line preservation、ease distribution），而differentiable方案主张**直接优化物理合体性**。这两种范式各有优劣：

- **GNN优势**：可以编码industry-specific规则（如特定品牌的推板传统）、离散尺码跳变更自然
- **Differentiable优势**：物理保真度高、可以处理任意体型（不限于标准尺码）、已有实际验证

Idea需要明确自己的positioning——是学习规则还是优化物理？如果是前者，与简单的rule-based系统相比，GNN的优势在哪里？

### 3. 多标准尺码系统是真正的differentiator——应重点强调

不同市场使用不同的尺码标准（GB/T 1335中国、ASTM D5585美国、EN 13402欧洲、JIS L4005日本），且这些标准的body measurement分布、尺码间距、关键测量部位都不同。现有方法（包括Inverse Garment和Dress Anyone）都不处理这种跨标准推板。如果GNN能学习不同标准的推板规则并在它们之间transfer，这才是真正无法被differentiable方案轻易替代的贡献。

### 4. 训练数据的可获取性是核心风险

工业推板数据是高度proprietary的——每个品牌的推板规则是核心商业机密。获取足够数量、足够多样性的训练数据可能是最大的瓶颈。Idea没有讨论数据来源策略：是合成数据？公开数据集？工业合作？此外，不同品牌/市场的推板规则差异极大——一个在运动服饰上训练的GNN可能完全不适用于正装。

### 5. 推板规则的可解释性需求

工业界对推板的一个核心要求是**可解释性**——版型师需要理解每个尺码的具体调整量及其原因（如"胸围增加4cm因为XL与L的胸围差为4cm"）。GNN作为黑箱模型，可能无法满足这一需求。AIDL（2025）的constraint specification方法或许是更适合工业场景的路径——将推板规则形式化为可验证的约束，而非学习隐式表示。

### 6. Grain line preservation和ease distribution是否需要GNN？

传统推板中，grain line preservation是硬约束（grain line必须保持垂直于布面），ease distribution遵循已知的工程规则。这些是否真的需要GNN来学习？还是说rule-based系统加上少量参数化调整就足够了？Idea需要论证GNN在这些方面的具体优势——是handling edge cases？是跨款式transfer？还是处理非线性推板关系？

## 改进建议

1. **承认并integrate先行工作**——将Inverse Garment的Pattern Linear Grading和Dress Anyone的automatic refitting作为baseline，而非忽略它们
2. **重新定位核心贡献**——从"第一个AI推板方法"改为"学习工业推板规则的GNN框架（vs物理优化方案）"
3. **突出多标准尺码系统**作为核心differentiator——这是现有方法都不处理的
4. **讨论数据获取策略**——合成数据生成（从已知规则生成训练样本）+ 少量工业数据fine-tuning
5. **增加可解释性模块**——GNN的推板结果应能输出human-readable的调整量和原因
6. **制定与Inverse Garment/Dress Anyone的对比实验计划**——在相同body shape变化上比较推板精度和物理合体性

## 评分调整建议

| 维度 | 原分 | 建议 | 理由 |
|------|------|------|------|
| Novelty (N) | 8 | 6 | 先行工作存在，"第一个"claim不成立 |
| Feasibility (F) | 7 | 6 | 训练数据获取是主要风险 |
| Impact (I) | 9 | 8 | 工业价值高但需要可解释性 |
| Confidence (C) | 8 | 7 | 需要与differentiable方案对比验证 |
| Evidence (E) | 6 | 6 | 不变 |
| Total | 38 | 33 | 方向有价值但需要重新定位和更扎实的baseline比较 |

## 补充参考文献

- **Inverse Garment** (2024): Section 3.3 Pattern Linear Grading — automated grading via control point relocation
- **Dress Anyone** (2024): Automatic refitting to arbitrary body shapes, 5-30min, real fabrication verified
- **AIDL** (2025): Constraint specification DSL — alternative to learning-based grading
- **GarmentLab**: Distinguishes PBD (large woven) vs FEM (small elastic) — relevant to grading simulation
- **DiffCloth**: Differentiable cloth simulation — potential foundation for physics-based grading
