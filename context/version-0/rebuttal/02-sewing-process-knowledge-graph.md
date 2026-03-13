<!-- markdownlint-disable -->
# Rebuttal: Sewing Construction Knowledge Graph

> Reviewer角色：跨领域审稿人（知识图谱 + 制造工艺）
> 审阅日期：2026-03-13
> 审阅对象：ideas/02-sewing-process-knowledge-graph

## 主要反驳意见

### 1. 相关工作遗漏严重，新颖性N=7可能虚高

Idea声称在服装领域构建缝制操作知识图谱是新颖的，但这忽略了多条已有工作线：首先，工业界的PLM（Product Lifecycle Management）系统（如Lectra Modaris、Gerber AccuMark）已经隐式编码了大量缝制工艺知识。其次，Textile IR（2025）已经提出了formal node types和七层验证架构的服装制造中间表示——这实质上就是一种结构化的领域知识编码方案。更重要的是，Textile IR不是"just a position paper"，而是提供了具体的节点类型定义和验证层次设计。Idea需要明确说明其KG与这些已有工作的差异化价值。

### 2. FALCON证明约束可直接嵌入生成过程

FALCON（2026）实现了100%可行性的代码生成，其核心机制是grammar-constrained decoding + repair + adaptive sampling。这意味着**缝制约束不一定需要显式的知识图谱来存储和查询**——它们可以被编码为生成时的语法约束，在解码过程中直接强制满足。这是一条根本性的替代路径：与其构建KG然后在生成时查询约束，不如将约束直接编译进生成器的采样空间。CAD-Assistant的tool augmentation（constraint checker使PF1从0.747提升到0.979）提供了另一个轻量方案：将约束检查作为工具调用，而非需要维护完整的知识图谱基础设施。

### 3. 知识图谱的长期维护成本被严重低估

这是知识图谱领域几十年来反复验证的难题：**构建KG容易，维护KG极难**。服装工艺并非静态知识——新材料（如3D打印面料、智能纺织品）、新工艺（超声波焊接、激光裁剪）、新标准持续涌现。KG的schema需要不断演化，实体关系需要更新，不一致性需要检测和修复。Idea中将可行性评为F=6，但实际的维护成本可能使长期可行性大打折扣。工业级KG项目（如Google Knowledge Graph、Wikidata）需要持续的大规模投入——一个学术项目能否承担这种持续成本？

### 4. 从"结构化知识"到"实际生成改进"的因果链不清晰

KG存储了"这种缝型需要1.5cm缝份"——然后呢？这条知识如何在版型生成pipeline中发挥作用？Idea的评分中Impact I=8，但缺少从知识存储到下游任务改进的具体机制设计。是作为prompt的context注入？作为constraint checker的规则库？还是作为reward model的信号源？每种路径的技术挑战截然不同。Chat2Layout的visual self-reflection机制表明，对于布局类任务，视觉反馈循环（OOB从24.8%降至21.0%）可能比纯符号化约束更有效。

### 5. 缝制操作的形式化面临领域特有的困难

与通用制造工艺不同，缝制操作的语义高度依赖上下文：同样是"平缝"，在薄纱上和在牛仔布上的参数、前处理、后整理完全不同。KG的节点和边是否能capture这种上下文依赖性？如果节点定义过粗，则知识无用；如果过细，则图谱爆炸性增长。这种粒度trade-off在Idea中没有被讨论。

### 6. 数据来源和质量控制机制缺失

构建KG需要大量领域知识标注。Idea计划从哪里获取这些数据？专家标注成本极高，自动抽取（从教材、手册）的质量难以保证。BetterBench的核心教训——"quality trumps scale"——直接适用：一个有错误的KG比没有KG更危险，因为它会以高置信度传播错误约束。

### 7. 与大语言模型的交互模式需要重新思考

LLM本身已经通过预训练隐式存储了大量缝制工艺常识。KG的增量价值在哪里？如果LLM已经"知道"平缝需要对齐边缘，KG的贡献是什么？SVGen的成功（3B模型超越GPT-4o）表明，在合适的训练策略（curriculum + CoT + GRPO）下，小模型可以学到非常精细的领域知识。KG的价值可能在于**LLM不知道的长尾知识和精确数值参数**——但这需要实验验证而非假设。

## 改进建议

1. **Hybrid方案**：核心安全约束用FALCON式grammar-constrained decoding硬编码（保证100%满足），长尾工艺知识用KG补充（提供建议性guidance）
2. **与Textile IR对标**：明确说明KG与Textile IR的seven-layer verification的关系——是补充、替代还是扩展？
3. **定义KG维护策略**：至少需要包括版本控制、自动不一致性检测、社区贡献机制
4. **设计具体的下游集成接口**：KG知识如何注入生成pipeline？给出至少两种方案的对比实验设计
5. **从小规模高质量起步**：先构建一个覆盖10种基础缝型的高精度KG，验证下游任务改进效果，再扩展

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 7 | 5 | Textile IR和工业PLM系统已有大量相关工作 |
| Feasibility | 6 | 5 | 长期维护成本和数据标注成本被低估 |
| Impact | 8 | 7 | 从KG到实际生成改进的因果链不清晰 |
| Clarity | 8 | 7 | 缺少与替代方案（FALCON、tool augmentation）的对比讨论 |
| Evidence | 7 | 6 | 缝制领域的已有结构化知识工作未被引用 |

## 补充参考文献

- **FALCON** (2026): Grammar-constrained decoding，100%可行性
- **Textile IR** (2025): Formal node types，seven-layer verification
- **CAD-Assistant** (2024): Tool-augmented VLLM，constraint checker PF1 0.747→0.979
- **BetterBench**: 数据质量 > 规模，MMLU教训
- **Chat2Layout**: Visual self-reflection，OOB 24.8%→21.0%
- **SVGen**: 3B模型超越GPT-4o，curriculum + CoT + GRPO训练策略
