<!-- markdownlint-disable -->
# Rebuttal: Hierarchical Layered Garment Generation

> Reviewer角色：跨领域审稿人（3D生成 + 服装仿真）
> 审阅日期：2026-03-13
> 审阅对象：ideas/07-multi-layer-garment-joint-generation

## 主要反驳意见

### 1. Dress-1-to-3已使用CIPC处理多层碰撞——"完全缺失"的假设不成立

Idea的一个核心假设是多层服装生成领域"完全缺失"可行方案。但Dress-1-to-3已经实现了基于CIPC（Continuous Intersection Prevention for Cloth）的多层碰撞处理闭环pipeline。虽然Dress-1-to-3主要处理的是3D draping而非2D版型生成，但其技术方案证明了多层服装的物理仿真和碰撞避免是可解决的问题。Idea应将Dress-1-to-3作为baseline进行对比，明确自己的增量贡献在版型层面（2D pattern generation）而非仿真层面（3D draping），否则新颖性声明将被削弱。

### 2. 训练数据是真正的瓶颈——Idea未充分讨论数据获取策略

多层服装生成的核心挑战不是模型架构，而是**数据**。目前不存在大规模的多层服装ground truth数据集——即同时提供内层+外层版型，并附带仿真后的3D穿着效果。GarmentCode数据集中的服装多为单层；SewFactory虽有一定规模，但多层组合极其有限。Idea中对数据问题的讨论不足以支撑其方案的可行性。需要回答：(a) 训练数据从哪来？(b) 需要多少数据？(c) 如何保证数据中的层间关系是物理上合理的？

### 3. 顺序条件生成可能比联合生成更实际

Idea提出hierarchical联合生成，但一种更简单的替代方案值得认真考虑：**顺序条件生成**——先生成内层服装，以内层为条件输入生成外层。这种方案有多个优势：(a) 降低了问题的组合复杂度，(b) 更符合人类设计师的实际工作流程（先穿内衣再搭配外套），(c) 训练数据需求更低（只需要"给定内层→合理外层"的配对，而非"所有层同时合理"的全局约束）。Chat2Layout的multi-turn editing方法论支持这种逐步条件化的思路——每一步添加一层，并通过visual self-reflection验证。

### 4. GarmentLab可提供训练数据生成pipeline——但规模化有挑战

GarmentLab（2025）的多物理仿真框架可以生成多层服装的仿真数据。其sim2real transfer在10/15任务上成功，这为数据生成提供了一条可行路径。但问题在于规模化：GarmentLab的每次仿真是否足够快以生成数万个训练样本？仿真的物理精度是否足以作为neural network的ground truth？如果训练数据的物理精度不够高，neural模型将学到错误的层间交互模式。

### 5. 层间交互的物理复杂度被低估

多层服装的物理行为远非"避免碰撞"这么简单。层间存在：(a) 摩擦力——内外层面料之间的相对滑动，(b) 空气层——保暖服装依赖层间空气的隔热效果，(c) 压缩效应——外层对内层的压力导致内层变形，(d) 面料粘连——某些面料组合会相互粘附。DiffCloth的可微仿真是否能准确捕捉这些细粒度的层间物理效应？大多数现有仿真器将层间交互简化为碰撞检测+摩擦力，忽略了上述(b)(c)(d)。

### 6. 评价指标的定义

如何评估多层服装生成的质量？单层服装可以用几何精度、缝合匹配度等指标。多层服装额外需要：层间间距合理性、碰撞次数/面积、整体轮廓的美学评价、运动时的相对位移。这些指标需要在方案设计阶段就定义清楚。N3D-VLM证明数值推理（92.1% vs 36.3%）在3D场景中对评估至关重要——多层服装的层间距离等数值属性需要专门的评估方法。

### 7. 现有工作的兼容性整合

Idea可以从多个现有方案中借鉴技术组件：Dress-1-to-3的CIPC碰撞处理、DiffAvatar的joint optimization framework、Inverse Garment的自动化grading。但这些方案各自有不同的版型表示、仿真器接口和优化框架——将它们整合进一个统一的多层生成pipeline本身就是一个非trivial的系统工程挑战。Idea的Feasibility F=5可能低估了这种系统集成的难度。

## 改进建议

1. **Dress-1-to-3作为baseline**：明确对比——Dress-1-to-3做3D draping，本Idea做2D pattern generation，贡献在不同层面
2. **顺序条件生成作为first step**：先实现"给定内层→生成外层"的条件生成，验证可行性后再扩展到联合生成
3. **GarmentLab数据pipeline设计**：详细规划如何使用GarmentLab生成训练数据——仿真参数、数据规模、质量验证流程
4. **分品类验证**：先聚焦一个简单场景（如T恤+夹克两层），再逐步增加层数和复杂度
5. **层间物理指标**：定义可计算的层间交互质量指标——碰撞面积、层间距离方差、运动稳定性

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 8 | 6 | Dress-1-to-3已证明多层碰撞处理可行，增量新颖性需聚焦于2D pattern层面 |
| Feasibility | 5 | 4 | 训练数据缺失是未解决的根本瓶颈 |
| Impact | 7 | 7 | 维持——多层服装是真实需求 |
| Clarity | 7 | 6 | 数据策略和层间物理模型讨论不足 |
| Evidence | 5 | 5 | 维持 |

## 补充参考文献

- **Dress-1-to-3**: CIPC多层碰撞处理，闭环pipeline
- **GarmentLab**: 多物理仿真框架，sim2real 10/15成功，3种transfer方案
- **DiffAvatar**: Joint pattern+body+material optimization
- **Inverse Garment**: 15x GPU加速，automated pattern grading
- **Chat2Layout**: Multi-turn editing，visual self-reflection
- **N3D-VLM**: 3D场景中的数值推理评估
- **DiffCloth**: 可微仿真——层间物理精度的局限性
