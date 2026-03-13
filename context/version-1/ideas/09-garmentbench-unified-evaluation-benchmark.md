<!-- markdownlint-disable -->
# Idea 09: GarmentBench 模块化评估框架（v1）

## v0→v1 变更说明

1. **架构重构：从单体benchmark转向模块化可组合框架**。采纳Rebuttal关于SAIBench SAIL DSL模式的建议，将GarmentBench从"大一统benchmark"重新设计为模块化评估框架，解耦任务定义（task）、模型接口（model adapter）和评估指标（metric module），使社区可独立贡献新组件。这一变更直接回应了"单体benchmark难以扩展"的批评。
2. **引入BetterBench质量保证体系**。根据Rebuttal指出的MMLU质量危机警告，整合BetterBench 46条标准（Reuel et al., 2024）和How2Bench 55条checklist（Cao et al., 2025）作为内建质量保证机制。新增statistical significance reporting、replication scripts、documentation completeness三项强制要求。
3. **显式difficulty stratification**。借鉴SVGenius的complexity stratification方法论（量化指标驱动的分层），将测试集从粗粒度的Simple/Medium/Complex三级扩展为基于量化复杂度指标的连续分层，并确保每个层级具备statistical power（≥50 test cases）。
4. **增加normative governance机制**。回应"benchmarks as normative instruments"的批评（Koch, 2025），新增Benchmark Governance Board设计，明确谁有权定义"好版型"标准，如何处理跨文化审美差异。
5. **新增anti-contamination和维护计划**。添加held-out test set + periodic refresh机制，以及versioning strategy、deprecation policy等长期维护路线图。
6. **评分下调**。新颖性从7降至6（benchmark方法论本身成熟，创新在于模块化+领域特化组合），可行性维持8（模块化实际上降低了初始构建门槛），影响力从10降至9（需审慎对待monoculture风险），清晰度维持9，证据支撑从8降至7（需更多领域内验证）。

## 概述

构建GarmentBench——AI版型生成领域的首个**模块化评估框架**，而非传统的单体benchmark。GarmentBench借鉴SAIBench的SAIL DSL思想，将评估系统解耦为三个独立可组合的层次：**任务描述层**（Task Description）、**模型适配层**（Model Adapter）和**指标评估层**（Metric Module）。这种设计使得自回归、扩散、程序合成、图像化等不同范式能够在统一接口下公平比较，同时允许社区在不修改核心框架的情况下贡献新任务、新指标和新数据集。

与NLP领域的GLUE/SuperGLUE不同，GarmentBench从设计之初就吸取了BetterBench揭示的benchmark质量危机教训——MMLU在46条质量标准中仅得5.5/15（Reuel et al., 2024），以及Koch（2025）关于"scientific monoculture"的警告。GarmentBench通过内建质量保证checklist、显式normative governance和模块化架构来避免重蹈覆辙。

## 指向的Gap

- **Gap 9: 范式间缺乏公平对比** — 四种根本不同的生成范式无法公平比较
- **Gap 27: 评估指标碎片化** — 每篇论文使用不同指标和数据集
- **Gap 5: 数据集偏差** — 提供多来源数据减轻单一合成引擎的偏差

## 推导过程

1. **观察**：SewingGPT用Panel L2，AIpparel用CLIP score，ChatGarment用user study——指标不可比
2. **NLP先例与警示**：GLUE benchmark (Wang et al., 2019) 通过统一评估推动了快速迭代，但其后继MMLU在BetterBench评估中仅得5.5/15——说明"建了benchmark"不等于"建了好benchmark"
3. **模块化先例**：SAIBench (Li & Zhan, 2022) 通过SAIL DSL解耦问题/模型/指标，证明模块化框架在跨学科benchmark中的优越性
4. **质量保证方法论**：BetterBench 46条标准 + How2Bench 55条checklist提供了系统性的质量保证方法论，可直接嵌入GarmentBench的设计流程
5. **版型领域的特殊性**：版型评估需要**多层级指标**——几何精度、拓扑正确性、物理可行性、穿着质量、可制造性——每个维度可能需要3-5个具体指标（SVGenius仅针对SVG生成就设计了18种指标），模块化架构允许各维度独立演进
6. **规范性反思**："好版型"的定义是文化、审美、功能的交汇——中国国标、意大利haute couture、日本和服pattern的评判标准根本不同。Benchmark必须显式声明其normative choices

## 详细设计

### 1. 模块化架构（核心变更）

**设计哲学**：受SAIBench SAIL DSL启发，GarmentBench采用三层解耦架构：

```
┌─────────────────────────────────────────────┐
│           GarmentBench Framework             │
├─────────────┬──────────────┬────────────────┤
│ Task Layer  │ Adapter Layer│ Metric Layer   │
│             │              │                │
│ • task.yaml │ • AR adapter │ • geometric.py │
│   (输入格式  │ • Diff adapt │ • topo.py      │
│    GT格式    │ • ProgSyn    │ • physics.py   │
│    难度标签)  │   adapter   │ • manufact.py  │
│             │ • Image adapt│ • perceptual.py│
└─────────────┴──────────────┴────────────────┘
```

**Task Description Language**：每个评估任务以YAML描述文件定义，包含：
- `input_modality`: 输入模态（text / image / sketch / parameters）
- `output_format`: 期望输出格式（panel_sequence / program / image）
- `ground_truth_schema`: GT数据schema
- `difficulty_level`: 量化复杂度分数
- `cultural_context`: 适用的文化/标准体系（可选）

**Model Adapter Protocol**：标准化接口，任何新范式只需实现：
- `generate(input) → output`: 生成接口
- `output_to_panels(output) → StandardPanelSet`: 输出转换为标准panel表示
- `metadata() → ModelInfo`: 模型元信息（参数量、推理时间等）

**Metric Module Interface**：每个指标模块独立实现：
- `compute(prediction, ground_truth) → score`: 计算分数
- `aggregate(scores) → summary`: 聚合统计
- `significance_test(scores_a, scores_b) → p_value`: 统计显著性检验（**强制要求**）

### 2. 测试集构成（改进版）

**规模**：600+ 服装样本，基于量化复杂度指标的连续分层

**Complexity Stratification**（借鉴SVGenius方法论）：

定义复杂度通过四个量化指标：
- **Panel Count** (PC): 片型数量
- **Curve Complexity** (CC): 边曲线的控制点总数
- **Stitch Density** (SD): 缝合关系数量/片型数量
- **Symmetry Breaking** (SB): 非对称结构比例

综合复杂度分数：`C = w₁·PC + w₂·CC + w₃·SD + w₄·SB`（权重通过专家标注校准）

| 难度层级 | 复杂度分数 | 样本数 | 品类示例 | Statistical Power |
|---------|-----------|--------|---------|-------------------|
| Level 1 (Easy) | C < 0.3 | ≥100 | A字裙、背心、直筒裤 | n≥50/sub-category |
| Level 2 (Moderate) | 0.3 ≤ C < 0.5 | ≥150 | T恤、衬衫、基础连衣裙 | n≥50/sub-category |
| Level 3 (Hard) | 0.5 ≤ C < 0.7 | ≥200 | 休闲裤、西装裤、A-line dress | n≥50/sub-category |
| Level 4 (Expert) | C ≥ 0.7 | ≥150 | 夹克、大衣、多层裙装、套装 | n≥50/sub-category |

**多模态输入**：每个样本配备
- 文本描述（专业版型术语 + 日常语言两个版本）
- 3D渲染图（正面、侧面、背面）
- 技术草图（flat sketch）
- 设计参数表

**Ground Truth**：
- 精确的panel几何（B-spline控制点）
- 完整的stitch关系
- 缝份宽度、标记点位置、Grain line方向
- 3D仿真渲染参考
- **文化标签**：标注该GT所依据的制版标准体系（如ISO/中国国标/日本JIS等）

### 3. 统一指标套件（模块化重构）

每个指标为独立模块，按层级组织。**Tier 1-2为必选**，Tier 3-5为推荐。

**Tier 1: 几何准确性**（必选，所有方法必须报告）
- **Panel Chamfer Distance (PCD)**: 预测panel与GT panel的Chamfer距离，归一化
- **Edge Curve Error (ECE)**: 边曲线的参数化拟合误差
- **Panel Count Accuracy (PCA)**: panel数量预测准确率
- **Panel Area Ratio (PAR)**: 预测panel面积与GT面积比（新增，检测scaling误差）

**Tier 2: 拓扑正确性**（必选）
- **Stitch F1 (SF1)**: 缝合关系的F1 score
- **Topology Graph Edit Distance (TGED)**: 预测的panel-stitch图与GT图的编辑距离
- **Grain Line Angular Error (GLAE)**: 布纹方向角度误差（新增）

**Tier 3: 物理可行性**（推荐）
- **Simulation Success Rate (SSR)**: 版型能否成功仿真到身体上（无穿透、无发散）
- **Drape Similarity (DS)**: 仿真悬垂效果与GT的相似度
- **Intersection Volume (IV)**: body-garment穿透体积（新增）

**Tier 4: 可制造性**（推荐）
- **Seam Allowance Score (SAS)**: 自动添加缝份的正确性
- **DXF Export Score (DES)**: 能否导出有效DXF文件
- **Fabric Utilization Rate (FUR)**: 排料利用率（新增）

**Tier 5: 感知质量**（推荐）
- **CLIP Similarity (CS)**: 渲染图与GT渲染图的CLIP余弦相似度
- **Human Preference Score (HPS)**: 人类盲审偏好评分
- **Style Consistency Score (SCS)**: 多视角渲染风格一致性（新增）

**统计要求**（BetterBench强制标准）：
- 所有指标报告**均值±标准差+95%置信区间**
- 模型间比较必须报告**paired t-test或Wilcoxon signed-rank test的p值**
- 每个难度层级单独报告结果

### 4. 评估协议（增强版）

- 固定数据划分（不允许在测试集上调参）
- 所有方法在相同硬件上报告推理时间
- **Blind test set**：20%测试数据不公开，仅通过API评估（anti-contamination）
- **Rolling refresh**：每12个月更新15-20%的blind test set（防止overfit）
- **Replication package**：每个baseline必须提供完整的replication scripts + Docker环境
- 在线排行榜，区分"必选指标排名"和"全指标排名"

### 5. Baseline实现

为四种主要范式提供PyTorch参考实现：

| 范式 | Baseline | 实现基础 |
|------|---------|---------|
| 自回归 | SewingGPT-base | 基于DressCode论文复现 |
| 扩散 | GarmentDiff-base | 基于GarmentDiffusion论文复现 |
| 程序合成 | GarmentCode-gen | 基于Design2GarmentCode简化版 |
| 图像化 | Garmage-base | 基于GarmageNet概念 |

每个baseline配备：
- 完整训练/推理代码 + Docker环境
- 在所有Tier 1-2指标上的结果及statistical significance报告
- 按难度层级的breakdown分析

### 6. Benchmark Governance（新增）

**Governance Board**组成：
- 版型设计专家（至少覆盖3个文化传统：西方、东亚、南亚）
- AI/ML研究者
- 工业界代表（服装制造商、CAD软件商）
- 社区贡献者代表

**职责**：
- 审批新增指标模块和任务定义
- 裁决normative disagreements（如不同文化对"好版型"的不同定义）
- 审核测试集更新和版本发布
- 发布年度"State of GarmentBench"报告

**Versioning Strategy**：
- Major version（如v2.0）：指标体系变更、任务定义变更
- Minor version（如v1.1）：测试集扩展、blind set refresh
- Deprecation policy：旧版本保留可访问性至少2年

### 7. Anti-Contamination设计（新增）

- **Held-out partition**：20%测试数据永不公开，仅通过评估API访问
- **Periodic refresh**：每12个月替换blind set中15-20%的样本
- **Canary tokens**：在测试集中嵌入少量故意设计的异常样本，监测数据泄露
- **Submission rate limiting**：每个team每月最多提交5次blind evaluation

## 参考文献分析

### 核心参考

1. **BetterBench** (Reuel et al., NeurIPS 2024)
   - 46条标准评估24个AI benchmark的质量
   - MMLU仅得5.5/15，14个benchmark在模型区分上无统计显著性
   - GarmentBench直接采用其checklist作为内建质量保证
   - **新增参考**

2. **Can We Trust AI Benchmarks?** (Koch et al., 2025)
   - Benchmark是normative instruments，定义什么是"好的"
   - 警告single benchmark导致"scientific monoculture"
   - GarmentBench通过模块化+governance回应此批评
   - **新增参考**

3. **SAIBench** (Li & Zhan, BenchCouncil 2022)
   - SAIL DSL解耦问题/模型/评估为可复用模块
   - 证明模块化benchmark框架在跨学科场景的优越性
   - GarmentBench的三层架构直接受其启发
   - **新增参考**

4. **How2Bench** (Cao et al., 2025)
   - 55条checklist覆盖code-related benchmark全生命周期
   - 发现70%的benchmark未进行数据质量保证
   - 为GarmentBench提供补充质量检查标准
   - **新增参考**

5. **SVGenius** (2025)
   - 仅针对SVG生成就设计了18种指标+3级复杂度分层
   - 证明单一视觉生成任务的评估复杂度远超预期
   - GarmentBench借鉴其complexity stratification方法论
   - **新增参考**

6. **GarmentLab** (Lu et al., NeurIPS 2024)
   - 服装操作的统一仿真和benchmark
   - 关注机器人操作而非版型生成，但其benchmark设计可供参考
   - **新增参考**

7. **GLUE Benchmark** (Wang et al., 2019)
   - 统一了9个NLP任务的评估，催生BERT→T5→GPT-3快速迭代
   - GarmentBench借鉴其多任务评估理念，但吸取其后继MMLU的质量教训

8. **SewingGPT/DressCode** (He et al., SIGGRAPH 2024)
   - 定义了Panel L2、Stitch F1等指标
   - GarmentBench将其纳入Tier 1/2并标准化

9. **GarmentDiffusion** (Li et al., IJCAI 2025)
   - 声称复现SewingGPT进行比较，但复现质量不透明
   - GarmentBench通过提供标准baseline + replication scripts消除复现偏差

10. **AIpparel** (Nakayama et al., CVPR 2025)
    - 使用CLIP score和FID——完全不同的指标体系
    - GarmentBench将CLIP归入Tier 5，与几何指标共存

11. **ChatGarment** (Bian et al., CVPR 2025)
    - 依赖user study评估——主观性强
    - GarmentBench将HPS纳入Tier 5但要求同时报告客观指标

12. **PeerBench / Benchmarking is Broken** (Cheng et al., 2025)
    - 提出proctored evaluation blueprint：sealed execution、item banking、delayed transparency
    - GarmentBench的blind test set + rolling refresh设计受此启发
    - **新增参考**

## 五维评分

| 维度 | v0分数 | v1分数 | 理由 |
|------|--------|--------|------|
| **新颖性** | 7/10 | 6/10 | Benchmark方法论本身成熟（BetterBench/SAIBench已系统化），创新在于模块化架构+领域特化指标+governance的组合，属于incremental innovation |
| **可行性** | 8/10 | 8/10 | 模块化架构实际上降低了初始构建门槛——可先实现Tier 1-2核心模块，再由社区贡献扩展；governance机制增加组织成本但提升可持续性 |
| **影响力** | 10/10 | 9/10 | 将改变领域评估方式，但需审慎对待monoculture风险；模块化设计允许多样化评估视角共存，部分缓解此风险 |
| **清晰度** | 9/10 | 9/10 | 模块化架构、指标层级、评估协议、governance均可精确定义；complexity stratification公式化可复现 |
| **证据支撑** | 8/10 | 7/10 | GLUE/ImageNet的成功先例+BetterBench/SAIBench的方法论支撑；但领域内尚无benchmark实践验证，模块化在小社区的adoption存在不确定性 |
| **总分** | **42/50** | **39/50** | 评分更审慎，反映了对benchmark methodology成熟度和adoption风险的深入理解 |

## 风险与挑战

1. **社区采纳**：模块化框架的价值取决于社区是否积极贡献新模块——garment AI社区规模较小，adoption动力可能不足。需要通过与SIGGRAPH/CVPR workshop合作、提供easy-to-use贡献指南来降低参与门槛
2. **Governance overhead**：跨文化Governance Board的组建和运营成本不低，需要平衡"足够代表性"和"决策效率"
3. **Normative tensions**：不同文化传统对"好版型"的定义冲突不可避免——模块化架构允许并行存在多套评估标准，但排行榜如何处理多标准排名是开放问题
4. **数据质量**：GT版型的精度直接影响评估公正性，需要严格的质量控制流程——BetterBench/How2Bench checklist提供了系统性保障
5. **范式差异**：不同范式的输出格式差异很大（程序 vs token序列 vs 图像），Model Adapter Protocol的设计需要足够通用又不丢失范式特色
6. **维护成本**：模块化降低了单点维护压力，但增加了兼容性维护负担（模块间版本兼容）
7. **Anti-contamination执行**：blind test set的安全性依赖于API访问控制和canary token监测——小社区中数据泄露的检测可能不够灵敏
