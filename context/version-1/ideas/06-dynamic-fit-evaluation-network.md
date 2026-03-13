<!-- markdownlint-disable -->
# Idea 06: 动态适体性评估网络 FitNet (v1)

## v0→v1 变更说明

本版本基于v0 Rebuttal的七项反馈进行了系统性修订：

1. **使用场景聚焦**：明确FitNet的核心价值在于大规模优化场景（evolutionary search的fitness评估、版型空间探索），而非单次设计评估。回应Rebuttal第2点关于可微仿真已足够快的质疑，将FitNet定位为**大批量筛选器**而非仿真替代品。
2. **Hybrid评估架构**：采纳Rebuttal建议，设计两层评估策略——FitNet做毫秒级初筛（filter），DiffCloth/PhysDrape对top-k候选做分钟级精确验证（verify）。这直接回应了GVU理论的Hallucination Barrier警告：neural verifier单独使用不可靠，但作为coarse filter与physics verifier级联使用时，系统整体可靠性由物理仿真保证。
3. **Uncertainty Quantification模块**：新增基于Monte Carlo Dropout + ensemble disagreement的不确定性估计，当输入接近训练分布边界时自动fallback到物理仿真。回应Rebuttal第1、7点关于OOD预测危险性和物理直觉缺失的批评。
4. **Sim-to-Real校准具体化**：参照GarmentLab的三种sim2real方案和Real-to-Sim fabric parameter estimation研究（PINN-based, DiffCloud, PhysNet），明确校准pipeline的每个环节。回应Rebuttal第4点。
5. **训练数据生成成本重估**：承认50K样本成本被低估，重新设计数据生成策略（active learning + curriculum sampling），降低所需样本量同时提升覆盖率。回应Rebuttal第3点。
6. **"合身度"多维定义**：引入层次化fit定义框架——物理层（ease、pressure、strain）、功能层（ROM自由度）、风格层（品牌/品类特定偏好），明确FitNet在物理层训练、风格层通过fine-tuning适配。回应Rebuttal第5点。
7. **输入表示重新设计**：参照PhysDrape的force-driven GNN和SNUG的自监督物理loss，重新设计输入编码方案。回应Rebuttal第6点关于CAD-LLaMA层次化表示的启示。

---

## 概述

训练一个快速神经代理模型（neural surrogate），从版型几何+身体参数直接预测穿着质量分数（合体性、舒适度、活动自由度），无需运行完整的物理仿真。FitNet的核心定位不是替代物理仿真，而是在**大规模版型空间搜索**中充当**毫秒级粗筛器**（coarse filter）：在evolutionary search、贝叶斯优化等需要数千次评估的场景中，FitNet先将候选版型从数千个缩减至数十个，再由物理仿真对top-k候选进行精确验证。这种**filter-then-verify**架构在保证最终精度的前提下，将总计算量降低1-2个数量级。

## 指向的Gap

- **Gap 28: 真实穿着舒适度评估的缺失** — 所有AI方法的评估指标都是纯几何或感知层面的，不评估穿着质量
- **Gap 23: 仿真-现实差距** — 通过学习真实穿着数据（而非仅仿真数据）可以部分弥补sim-to-real gap
- **Gap 26: 生成-仿真闭环的缺失** — FitNet作为快速评估器使闭环优化在计算上可行

## 推导过程

### 从Gap到Idea的逻辑链

1. **闭环的核心障碍是规模而非单次速度**：v0的论述聚焦于"仿真太慢"，但Rebuttal正确指出DiffCloth（85x加速）和Dress Anyone（5-30分钟pipeline）已使单次仿真达到可接受水平。真正的瓶颈是**大规模搜索**：当版型优化需要评估数千个候选（如evolutionary pattern search、diversity-driven exploration）时，即使每次仿真仅需2分钟，2000个候选仍需约67小时。FitNet将此压缩至~20秒初筛 + ~1小时精确验证（top-30）。

2. **Neural surrogate作为coarse filter的成熟范式**：在药物发现（virtual screening → wet-lab validation）、芯片设计（fast surrogate → SPICE verification）、分子动力学（neural potential → DFT refinement）等领域，"快速粗筛+精确验证"是标准工作流。版型评估完全适合这一范式。

3. **自监督物理loss的突破降低数据需求**：SNUG（Santesteban et al., CVPR 2022）和Neural Cloth Simulation（Bertiche et al., TOG 2022）证明，通过将物理方程（膜应变能、弯曲能、碰撞惩罚、惯性项）直接作为训练loss，神经网络可以**无需ground-truth仿真数据**学习布料变形。PhysDrape（2025）进一步将force-driven GNN与可学习物理solver结合，实现端到端自监督训练。这些进展意味着FitNet不必完全依赖预计算的仿真数据集——可以采用**物理信息自监督**预训练 + 仿真数据fine-tuning的混合策略。

4. **Real-to-Sim研究提供了校准方法论**：最新的fabric real2sim研究（2025）系统评估了四类参数估计方法（DiffCloud差分管线、DiffCP、PhysNet数据驱动、PINN物理信息网络），发现PINN在准静态任务中表现最优但在动态场景受限，而仿真引擎选择和real2sim场景选择对性能有显著影响。这为FitNet的校准策略提供了具体的方法论指导。

## 详细设计

### 1. 两层评估架构（Filter-then-Verify）

```
版型候选池 (N=2000)
    │
    ▼
┌─────────────────────┐
│  Layer 1: FitNet    │  ~10ms/sample → 20s total
│  (neural surrogate) │  输出: fit score + uncertainty σ
└─────────────────────┘
    │
    │ 按 fit_score 排序，取 top-k (k=30)
    │ 对 σ > threshold 的样本标记为 "需物理验证"
    ▼
┌─────────────────────────────┐
│  Layer 2: Physics Verifier  │  ~2min/sample → 60min total
│  (DiffCloth / PhysDrape)    │  输出: precise fit metrics
└─────────────────────────────┘
    │
    ▼
最终 top-5 候选版型
```

**关键设计决策**：
- FitNet的误判方向应偏向**false positive**（不应遗漏优秀版型），通过调整ranking threshold实现
- 当FitNet的uncertainty σ超过阈值时，即使fit_score较低也强制进入Layer 2验证
- Layer 2的物理仿真结果反馈用于FitNet的在线fine-tuning（continual learning）

### 2. FitNet架构（重新设计）

```
输入层：
  - 版型表示: 层次化编码（Hierarchical Pattern Encoding）
    Level 1: 全局拓扑 — panel数量、连接关系、对称性 (graph-level)
    Level 2: panel几何 — B-spline控制点 (P × C × 2)
    Level 3: 缝合约束 — stitch pairs + seam allowance (S × 4)
  - 身体参数: SMPL-X shape coefficients (β ∈ R^10) + 关键部位尺寸 (∈ R^20)
  - 面料属性: (stretching_stiffness, bending_stiffness, density, friction)

编码器（Physics-Informed）:
  - Pattern Encoder: Force-Driven GNN（受PhysDrape启发）
    - 节点特征: panel顶点坐标 + 局部曲率 + 面积权重
    - 边特征: panel内边（膜约束）+ 跨panel stitch边（缝合约束）
    - 消息传递: 物理力传播模式（而非纯数据驱动）
  - Body Encoder: MLP on SMPL-X β → 关键部位3D坐标
  - Fabric Encoder: MLP on material parameters

融合（Cross-Modal Attention）:
  - body关键部位坐标 attend to pattern节点特征
  - 建模"哪个版型区域覆盖哪个身体部位"的对应关系
  - 输出: body-pattern alignment features (∈ R^K×D)

预测头:
  - Ease Predictor: 各部位ease预测 (∈ R^K, K=20个身体关键部位)
  - Pressure Predictor: 各部位压力预测 (∈ R^K)
  - Comfort Score: 综合舒适度评分 (∈ [0, 1])
  - Drape Quality: 悬垂质量评分 (∈ [0, 1])
  - Violation Detector: 穿透/过紧标志 (binary per region)

不确定性头:
  - MC Dropout: 推理时多次前向传播，计算预测方差
  - Ensemble Disagreement: 3个独立训练的FitNet子网络的预测分歧度
  - 输出: σ_ease, σ_comfort, σ_overall ∈ R^+
```

### 3. 训练策略（混合方案）

**Phase 1: 物理信息自监督预训练**（无需仿真数据）

受SNUG和Neural Cloth Simulation启发，用物理能量函数作为训练loss：
```
L_pretrain = L_membrane + L_bending + L_collision + L_gravity

其中:
  L_membrane: 膜应变能（面料拉伸不超过材料极限）
  L_bending:  弯曲能（悬垂合理性）
  L_collision: 身体-布料穿透惩罚
  L_gravity:  重力作用下的平衡约束
```
此阶段学习的是物理先验，不需要仿真ground truth。

**Phase 2: 仿真数据监督Fine-Tuning**

改进的数据生成策略（回应Rebuttal对50K成本的批评）：
```
初始种子集: 3种品类 × 100体型 × 10变体 = 3,000样本（约4天仿真）
    │
    ▼
Active Learning循环 (5轮):
  - FitNet预测当前未标注池的fit scores + uncertainty
  - 选择uncertainty最高的500个样本进行仿真
  - 用新数据fine-tune FitNet
  - 重复
    │
    ▼
最终有效训练集: ~5,500个高信息量样本
（等效覆盖率 ≈ 传统随机采样50K样本的覆盖率）
```

**Phase 3: Real-World校准**（可选）

参照real2sim fabric研究的方法论：
```
校准数据源:
  - 3D body scan + 真实穿着扫描 (10-20个样本/品类)
  - 压力传感垫 (pressure mapping) 数据
  - 动作捕捉下的面料动态行为

校准方法选择 (按推荐优先级):
  1. PINN-based: 用弹性动力学PDE约束网络，在准静态fit评估中表现最优
  2. DiffCloud差分管线: 端到端梯度优化，适合已有可微仿真器的场景
  3. PhysNet数据驱动: 基于triplet contrastive loss的物理相似度匹配

校准策略:
  - 对FitNet最后两层进行domain adaptation
  - 保持physics-informed预训练权重冻结
  - 校准后精度指标: ease预测MAE < 5mm, comfort score correlation > 0.85
```

### 4. "合身度"层次化定义框架

回应Rebuttal第5点（"合身度"定义本身就是研究问题）：

| 层次 | 指标 | 计算方式 | FitNet角色 | 品牌/品类依赖性 |
|------|------|---------|-----------|---------------|
| **物理层** | Ease分布 | body-cloth间距 | 直接预测 | 低（物理量，客观） |
| **物理层** | 压力分布 | cloth stress在body表面投影 | 直接预测 | 低 |
| **物理层** | 穿透量 | body-cloth intersections | 直接预测 | 无 |
| **功能层** | ROM自由度 | 关键动作下的面料应变 | 动态pose下预测 | 中（运动类型相关） |
| **功能层** | 悬垂系数 | drape area / fabric area | 直接预测 | 中 |
| **风格层** | 品牌fit偏好 | 目标ease profile | 用户定义目标 | 高（品牌特定） |
| **风格层** | 文化fit标准 | 地区/文化特定tolerance | 用户定义目标 | 高 |

**设计决策**：FitNet在物理层和功能层进行**通用训练**，输出客观的物理量预测。风格层通过**用户定义的目标ease profile**实现，FitNet不需要"理解"什么是"合身"——它只需准确预测物理量，由上层优化器根据品牌特定的目标函数进行搜索。

### 5. 作为闭环优化的奖励函数

```python
def evolutionary_pattern_search(base_pattern, body, target_fit, population=2000):
    """FitNet驱动的进化版型搜索"""
    # 初始化种群
    candidates = generate_variations(base_pattern, N=population)

    for generation in range(50):
        # Layer 1: FitNet快速筛选
        scores, uncertainties = FitNet.predict_with_uncertainty(candidates, body)
        ranking = rank_candidates(scores, target_fit)

        # 选择top-k + 高不确定性样本
        top_k = ranking[:30]
        uncertain = [c for c, u in zip(candidates, uncertainties) if u > threshold]
        to_verify = deduplicate(top_k + uncertain)

        # Layer 2: 物理仿真精确验证
        verified_scores = physics_simulate(to_verify, body)

        # 用验证结果更新FitNet (online learning)
        FitNet.update(to_verify, verified_scores)

        # 进化操作
        parents = select_best(verified_scores, k=10)
        candidates = crossover_and_mutate(parents, N=population)

    return select_best(verified_scores, k=5)
```

**速度对比（修正后）**：
- 纯物理仿真搜索: 2000候选 × 2min = ~67小时
- FitNet filter-then-verify: 20s筛选 + 30候选 × 2min = ~61分钟
- **实际加速比: ~66x**（比v0声称的6000x保守但更诚实）

## 参考文献分析

### 核心参考

1. **SNUG** (Santesteban et al., CVPR 2022)
   - 自监督学习动态3D服装变形，无需ground-truth仿真数据
   - 将物理方程（膜应变能、弯曲能、碰撞惩罚）转化为优化目标
   - 训练时间较监督方法快两个数量级
   - **对FitNet的启示**: Phase 1物理信息自监督预训练的理论基础

2. **Neural Cloth Simulation** (Bertiche et al., TOG 2022)
   - 首个能够无监督学习真实布料动力学的方法
   - 自动解耦静态和动态布料子空间
   - 提出运动增强技术提升泛化能力
   - **对FitNet的启示**: 静态/动态fit解耦——静态pose评估和动态运动评估应分别建模

3. **PhysDrape** (2025)
   - Force-driven GNN + 可学习物理solver的端到端自监督框架
   - 力作为中介连接神经网络和物理变形模型
   - 在CLOTH3D上实现更低能量分数、更少穿透
   - **对FitNet的启示**: Pattern Encoder采用force-driven GNN设计，使编码包含物理语义

4. **Dress Anyone** (2025)
   - 基于可微仿真的自动版型refitting
   - 5-30分钟完成从设计到制造的完整pipeline
   - 使用XPBD + 可微梯度优化2D sewing pattern
   - **对FitNet的启示**: Layer 2 physics verifier的实现基础

5. **GarmentLab** (Lu et al., NeurIPS 2024)
   - 统一仿真和benchmark，包含FEM和PBD两种物理方法
   - 三种sim2real方案，15个任务中10个成功迁移
   - **对FitNet的启示**: sim2real校准的具体方法论参照

6. **Real-to-Sim Fabric Study** (2025)
   - 系统评估DiffCloud、DiffCP、PhysNet、PINN四种real2sim方法
   - PINN在准静态任务最优，但动态场景受限
   - 仿真引擎和场景选择对性能有显著影响
   - **对FitNet的启示**: Phase 3校准方法选择的实证依据

7. **Wang et al. (2021) — PNN for Garment Fit**
   - 概率神经网络预测3D虚拟环境中的服装合体等级
   - 从ease allowance、数字压力、面料力学性能预测fit level
   - 对比多种模型（LR, SVM, RBF-ANN, BP-ANN），PNN精度最优
   - **对FitNet的启示**: 直接先例——输入特征设计和评估标准可参照

8. **Liu et al. (2023) — BP-ANN for Garment Fit Evaluation**
   - 无需试穿即可评估服装合体性的BP神经网络模型
   - 输入为人体测量数据+版型数据+面料属性
   - 284个实验样本训练，真实试穿验证有效
   - **对FitNet的启示**: 证实"版型+身体+面料→fit"映射的可学习性，但规模和表示需升级

### GVU理论的回应

Rebuttal正确指出GVU理论的Hallucination Barrier对neural verifier的警告。v1的回应：
- FitNet**不单独作为verifier**，而是作为**coarse filter**与physics verifier级联
- 最终决策由Layer 2物理仿真做出，FitNet的hallucination不影响最终结果
- Uncertainty quantification提供了**何时信任neural prediction**的定量依据
- 这一设计模式类似GVU推荐的"weaker model generates, stronger model verifies"——此处FitNet是weaker/faster model，物理仿真是stronger/slower model

## 五维评分

| 维度 | v0分 | v1分 | 理由 |
|------|------|------|------|
| **新颖性** | 8 | 7 | Neural surrogate在其他物理域已有先例（药物发现、分子动力学）；在服装版型评估域有少量直接先驱（Wang 2021, Liu 2023），但FitNet的filter-then-verify架构+uncertainty-aware fallback+physics-informed预训练的组合在版型域是新的 |
| **可行性** | 6 | 6 | Active learning策略将数据需求从50K降至~5.5K；物理信息自监督预训练降低了对仿真数据的依赖；但OOD泛化和多品类扩展仍是挑战。维持6分——挑战从数据量转移到了模型精度 |
| **影响力** | 8 | 8 | 维持——filter-then-verify架构使大规模版型搜索从理论可能变为实际可行。66x加速虽不如v0声称的6000x惊人，但在evolutionary search场景中仍是变革性的 |
| **清晰度** | 7 | 8 | 两层评估架构清晰定义了FitNet的角色和边界；"合身度"层次化定义解决了v0中"fit指标模糊"的问题；训练pipeline三阶段设计具体可执行 |
| **证据支撑** | 5 | 6 | 新增直接先驱（Wang 2021 PNN、Liu 2023 BP-ANN证实fit可学习性）、自监督物理训练的理论基础（SNUG、Neural Cloth Simulation）、real2sim校准的实证方法论（GarmentLab、real2sim fabric study） |
| **总分** | **34/50** | **35/50** | |

## 风险与挑战

1. **OOD泛化仍是核心风险**：尽管增加了uncertainty quantification和fallback机制，但FitNet对训练分布外的版型（极端尺码、非常规拓扑）的预测精度仍不确定。**缓解**: active learning循环持续扩展训练分布。

2. **物理信息自监督的精度上限**：SNUG和Neural Cloth Simulation的自监督方法在布料变形预测上已达到较好精度，但**fit指标预测**（需要body-cloth interaction的全局信息）比**vertex position预测**更为抽象，自监督精度是否足够需实验验证。

3. **多pose评估的计算开销**：静态pose下的fit评估相对简单，但Rebuttal第7点(DeepPHY)正确指出动态pose（弯腰、举手）更为关键。多pose评估会增加FitNet的推理次数（每pose一次），可能削弱速度优势。**缓解**: 定义3-5个canonical poses（站立、坐下、举手、弯腰、行走），对每个候选做multi-pose评估。

4. **Active learning的冷启动问题**：Phase 2的active learning依赖FitNet已有一定预测能力来选择informative样本。Phase 1的物理信息预训练能否提供足够好的初始化需验证。

5. **品牌/文化差异的long-tail**：物理层fit指标是通用的，但风格层的品牌fit偏好分布极度long-tail。FitNet不尝试解决此问题——将其留给上层优化器的目标函数定义。

## 分阶段验证路线图

| 阶段 | 目标 | 成功标准 | 预估时间 |
|------|------|---------|---------|
| **Stage 0** | 单品类（T恤）+ 静态pose原型 | ease预测MAE < 10mm, ranking correlation > 0.8 | 4周 |
| **Stage 1** | Active learning数据生成 + 多体型泛化 | 覆盖BMI 18-35范围，ease MAE < 8mm | 3周 |
| **Stage 2** | 多pose扩展（5 canonical poses） | 动态fit指标correlation > 0.75 | 3周 |
| **Stage 3** | 多品类扩展（裤装、连衣裙） | 跨品类ranking correlation > 0.7 | 4周 |
| **Stage 4** | Real-world校准 + 端到端pipeline集成 | 真实试穿验证，comfort score correlation > 0.7 | 6周 |
