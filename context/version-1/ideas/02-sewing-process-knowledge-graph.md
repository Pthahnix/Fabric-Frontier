<!-- markdownlint-disable -->
# Idea 02: 缝纫工艺知识图谱（v1）

## v0→v1 变更说明

| 变更项 | v0 内容 | v1 修订 | 依据 |
|--------|---------|---------|------|
| **定位重构** | 独立的知识图谱基础设施 | Hybrid架构：grammar-constrained硬约束 + KG软约束的分层系统 | Rebuttal §2: FALCON证明约束可直接嵌入生成过程 |
| **与Textile IR关系** | 仅称"互补" | 明确定义为Textile IR七层验证架构的**工艺语义扩展层**，复用其PatternPiece/SeamEdge节点类型 | Rebuttal §1: 与Textile IR对标不充分 |
| **粒度策略** | 未讨论 | 引入Context-Dependent Parameter Resolution机制，解决同一缝型在不同面料上参数不同的问题 | Rebuttal §5: 上下文依赖性导致粒度trade-off |
| **维护策略** | 仅列为风险 | 新增版本控制、LLM辅助知识抽取、社区贡献三重维护机制 | Rebuttal §3: 长期维护成本被严重低估 |
| **下游集成** | 笼统的"后处理"或"条件注入" | 具体化为三条集成路径并给出对比实验设计 | Rebuttal §4: 从KG到生成改进的因果链不清晰 |
| **与LLM关系** | 未讨论 | 明确KG的增量价值在于长尾知识和精确数值参数，而非LLM已知的常识 | Rebuttal §7: LLM已隐式存储大量工艺常识 |
| **新颖性评估** | N=7 | 下调至N=6，承认PLM系统和Textile IR的先行工作 | Rebuttal §1; 新增文献 |
| **启动策略** | 全量构建 | 采用"10种基础缝型"的最小可行KG验证路径 | Rebuttal改进建议§5 |
| **新增文献** | — | Wen et al. 2023 (制造KG系统化建模)、He & Jiang 2019 (制造KG连接主义)、纺织领域KG构建方法、Textile IR七层验证细节 | 学术搜索补充 |

## 概述

构建一个**混合约束架构（Hybrid Constraint Architecture）**，将缝纫工艺知识分为两类进行差异化编码：

1. **硬约束（Hard Constraints）**：安全关键的、必须100%满足的制造规则（如缝份不能为负值、缝合边长度必须匹配），采用grammar-constrained decoding直接编译进生成器的采样空间，借鉴FALCON的100%可行性机制。
2. **软约束（Soft Constraints）**：建议性的、依赖上下文的工艺知识（如"法式缝适合薄面料"、"袖窿notch应放置在肩点前后各1cm"），存储在**缝纫工艺知识图谱（Sewing Construction Knowledge Graph, SCKG）**中，作为可查询的guidance层。

SCKG与Textile IR的七层验证架构（Teikari & Fuenmayor, 2026）形成明确的分工关系：Textile IR定义节点类型（PatternPiece, SeamEdge, Dart, Notch, MaterialRegion）和几何/物理验证层（Layer 1-5），SCKG在此之上扩展**工艺语义层**——编码"用什么缝法"、"缝份多宽"、"标记点放哪里"等制造工艺知识，对应Textile IR的Layer 6b（制造可行性验证）。

## 指向的Gap

- **Gap 3: 缝纫语义的缺失** — 现有方法仅编码"哪些边缝在一起"（stitch tags），不区分缝合工艺类型（平缝、包缝、暗缝等），也不编码工艺参数
- **Gap 17: 缝份（Seam Allowance）的自动生成** — 缝份宽度因部位、工艺、面料而异，现有AI系统无法整合这些规则
- **Gap 18: 标记点（Notch）和对位点的缺失** — 工业版型需要notch、grain line、dart point等标记
- **Gap 19: 面料约束的缺失** — 面料特性直接决定多个版型参数（缝份宽度、缝合方式、grain line方向等）

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察Gap 17-19的共性**：缝份、标记点、面料约束这三个Gap的共同根源是——AI版型生成方法**缺乏服装制造工艺知识**。DressCode (He et al., 2024)的stitch tags仅编码3D坐标中点，GarmentDiffusion等方法输出panel顶点坐标 + edge参数 + stitch tags，均无缝份、无标记点、无工艺信息。

2. **知识表示的双轨需求**：分析这些工艺知识的特性，发现它们天然分为两类：
   - **确定性规则**（如"缝合边长度必须匹配"、"缝份宽度 > 0"）→ 适合编译为grammar constraint
   - **上下文依赖的启发式知识**（如"法式缝适合薄面料但不适合厚面料"、"格纹面料的裙片grain line必须经向"）→ 适合知识图谱的关系表示和推理

3. **Hybrid架构的必然性**：FALCON (2026)证明grammar-constrained decoding可实现100%可行性，但其适用范围限于可形式化为context-free grammar的确定性规则。服装工艺中大量启发式、统计性、上下文依赖的知识无法简单编码为grammar——因此需要KG作为补充。CAD-Assistant (2024)的tool augmentation（constraint checker使PF1从0.747提升到0.979）提供了另一种参考：将约束检查作为工具调用。

4. **与Textile IR的互补定位**：Textile IR已定义了PatternPiece、SeamEdge、Dart、Notch、MaterialRegion等节点类型和七层验证架构，但其关注点在几何有效性和物理仿真，**不涉及具体的工艺选择和参数决策**。SCKG恰好填补这一语义空白。

### 与现有工作的关系

| 已有工作 | 其贡献 | SCKG的差异化 |
|----------|--------|-------------|
| **Textile IR** (Teikari & Fuenmayor, 2026) | 七层验证架构、节点类型定义、双向信息流 | SCKG扩展Layer 6b，编码工艺选择规则而非仅验证几何有效性 |
| **工业PLM系统** (Lectra Modaris, Gerber AccuMark) | 隐式编码缝制工艺知识于专有软件中 | SCKG将隐式知识显式化、开放化、可查询化 |
| **制造知识图谱** (Wen et al., 2023; He & Jiang, 2019) | 通用制造领域的KG建模方法 | SCKG针对服装缝制的特有挑战：上下文依赖性、面料-工艺交互、隐性经验知识 |
| **FALCON** (2026) | Grammar-constrained decoding实现100%可行性 | SCKG处理FALCON无法覆盖的软约束和启发式知识 |
| **CAD-Assistant** (2024) | Tool-augmented VLLM, constraint checker | SCKG提供结构化的领域知识库而非通用CAD工具 |
| **ASTM D6673 / ISO 4916** | 数字版型和缝合类型标准 | SCKG是标准的"语义扩展"——不仅存储格式，还存储工艺规则 |
| **GarmentCode DSL** (Korosteleva et al., 2023) | 通过程序语义保证几何有效性 | SCKG在几何有效性之上叠加制造工艺有效性 |

## 详细设计

### 1. 混合约束架构

```
┌────────────────────────────────────────────────┐
│              AI版型生成模型                      │
│  (GarmentDiffusion / SewingGPT / ChatGarment)  │
├────────────────────────────────────────────────┤
│  Layer A: Grammar-Constrained Decoding (硬约束) │
│  ┌──────────────────────────────────────┐      │
│  │ • 缝合边长度匹配 (Δl < ε)           │      │
│  │ • 缝份宽度 > 0                       │      │
│  │ • Panel闭合性 (首尾顶点重合)         │      │
│  │ • Grain line角度 ∈ {0°, 45°, 90°}    │      │
│  │ • 面料幅宽约束 (panel宽 ≤ 幅宽)      │      │
│  └──────────────────────────────────────┘      │
│           ↓ 100%满足                            │
├────────────────────────────────────────────────┤
│  Layer B: SCKG Query (软约束/建议)              │
│  ┌──────────────────────────────────────┐      │
│  │ • 缝合类型推荐 (基于部位+面料)       │      │
│  │ • 缝份宽度精确值 (基于缝型+部位)     │      │
│  │ • Notch放置位置 (基于缝型+曲率)      │      │
│  │ • 工艺序列建议 (基于依赖关系)        │      │
│  │ • 面料-工艺兼容性检查                │      │
│  └──────────────────────────────────────┘      │
│           ↓ 最佳实践建议                        │
├────────────────────────────────────────────────┤
│  Layer C: Textile IR Verification Ladder        │
│  (Layer 1-5: 几何+物理验证)                     │
│  (Layer 6b: 制造可行性 ← SCKG提供规则)         │
└────────────────────────────────────────────────┘
```

### 2. 知识图谱本体设计

**2.1 实体类型（Entity Types）**

| 实体类型 | 示例 | 关键属性 |
|----------|------|----------|
| SeamType | 平缝、包缝、暗缝、法式缝、flat-felled seam | 适用面料范围、所需设备、缝份需求范围、强度等级、难度等级 |
| HemFinish | 单折边、双折边、窄卷边、包边、粘合边 | 折边宽度范围、适用位置、面料要求 |
| ClosureType | 拉链、纽扣、暗扣、魔术贴、绳扣 | 安装位置约束、缝份需求、面料兼容性 |
| FabricType | 棉布、丝绸、牛仔、针织、雪纺 | 弹性模量、克重、厚度、grain方向性、KES参数映射 |
| PatternRegion | 领口、肩缝、侧缝、下摆、袖窿、腰线 | 曲率特征、受力方向、美观优先级 |
| MarkType | V-notch、T-notch、drill hole、grain line、dart point | 形状规范、尺寸范围、适用位置 |
| OperationStep | 裁剪、缝合、熨烫、整理 | 前置条件、设备要求、质量检查点 |

**2.2 关系类型（Relation Types）**

| 关系 | 签名 | 示例 | 语义 |
|------|------|------|------|
| requires_seam_allowance | (SeamType, PatternRegion, FabricType) → width_range | (平缝, 侧缝, 棉布) → [0.8, 1.2]cm | **三元上下文依赖**的缝份规则 |
| compatible_with | (SeamType, FabricType) → {score, notes} | (法式缝, 雪纺) → {0.95, "推荐"} | 工艺-面料兼容性评分 |
| notch_placement | (SeamType, PatternRegion) → position_rules | (设袖缝合, 袖窿) → [肩点, 前标记, 后标记] | 标记点放置规则 |
| grain_direction | (PatternRegion, FabricType) → angle_constraint | (裙片, 格纹面料) → 0°±2° | grain line方向规则 |
| follows | (OperationStep, OperationStep) → sequence_constraint | (缝合侧缝, 折下摆边) → must_precede | 工艺序列约束 |
| excludes | (SeamType, FabricType) → reason | (法式缝, 牛仔) → "厚度过大导致层叠困难" | 互斥关系 |

**2.3 Context-Dependent Parameter Resolution**

v0的一个核心缺陷是未处理上下文依赖性——同一SeamType在不同FabricType上的参数完全不同。v1引入**三元关系查询**机制：

```
QUERY: seam_allowance(SeamType=平缝, Region=侧缝, Fabric=?)
  → Fabric=棉布:   1.0cm ± 0.2cm
  → Fabric=丝绸:   0.8cm ± 0.1cm  (注: 需配合包边处理)
  → Fabric=牛仔:   1.5cm ± 0.3cm  (注: 需flat-felled finish)
  → Fabric=针织:   1.0cm ± 0.2cm  (注: 需弹性缝线)
```

当查询的三元组在KG中无精确匹配时，采用**层次回退策略**：
1. 精确匹配：(SeamType=X, Region=Y, Fabric=Z)
2. 面料类别回退：(SeamType=X, Region=Y, FabricCategory=Z.parent)
3. 部位类别回退：(SeamType=X, RegionCategory=Y.parent, FabricCategory=Z.parent)
4. 统计默认值：基于SeamType的全局参数分布

### 3. 知识来源与构建方法

**3.1 最小可行KG（MVP-KG）：10种基础缝型起步**

遵循"小规模高质量"原则，首先构建覆盖以下10种基础缝型的高精度KG：

| 缝型 | ISO 4916编号 | 覆盖场景 |
|------|-------------|----------|
| 平缝 (Plain seam) | SSa-1 | 通用缝合 |
| 法式缝 (French seam) | SSa-2 | 薄面料精细处理 |
| Flat-felled seam | SSb-1 | 牛仔/工装 |
| 包缝 (Overlock) | SSa-1+EFb | 边缘处理 |
| 暗缝 (Blind hem) | BSa | 下摆处理 |
| 嵌条缝 (Welt seam) | LSa | 口袋 |
| 双针缝 (Twin needle) | SSa-2 | 针织面料 |
| 拉链缝 (Zipper seam) | SSa+附件 | 闭合结构 |
| 贴边缝 (Facing seam) | SSa+翻折 | 领口/袖口 |
| 褶裥缝 (Dart seam) | — | 立体成型 |

**3.2 知识来源分层**

| 层级 | 来源 | 方法 | 质量控制 |
|------|------|------|----------|
| **Tier 1: 标准文档** | ISO 4916、ASTM D6673、各国缝制标准 | 手工提取+结构化编码 | 双人交叉验证 |
| **Tier 2: 教科书** | Aldrich《Metric Pattern Cutting》、Armstrong《Patternmaking》 | LLM辅助抽取 + 专家审核 | 专家审核通过率 ≥ 95% |
| **Tier 3: 工业数据** | DXF文件中的缝份宽度分布、标记点模式 | 统计分析 + 异常检测 | 样本量 ≥ 100件/缝型 |
| **Tier 4: 专家知识** | 版型师访谈、隐性经验 | 结构化访谈 + 知识卡片法 | 多专家一致性 ≥ 80% |

**3.3 LLM辅助知识抽取pipeline**

LLM本身已通过预训练隐式存储了大量缝制工艺常识（Rebuttal §7）。SCKG的增量价值不在于重复LLM已知的常识，而在于：
- **长尾知识**：特定品牌规范、地区标准差异、新工艺参数
- **精确数值参数**：具体的缝份宽度、notch间距、grain line角度容差
- **可审计性**：每条知识有来源、版本、置信度标注，可追溯可验证

Pipeline：
```
教科书/手册文本 → LLM结构化抽取 (entity, relation, attribute)
                → 专家审核 (accept/reject/modify)
                → KG写入 (with provenance metadata)
                → 自动不一致性检测 (冲突规则标记)
```

### 4. 三条下游集成路径

针对Rebuttal §4的批评，明确设计三条从"知识存储"到"生成改进"的具体路径：

**路径A：Constraint Checker Tool（工具调用模式）**

借鉴CAD-Assistant的tool augmentation范式：
```
生成模型输出panel geometry + stitch tags
    ↓
Tool Call: SCKG.check_constraints(panels, stitches, fabric_type)
    ↓
返回: {violations: [...], suggestions: [...], confidence: 0.87}
    ↓
生成模型根据feedback修正输出（self-reflection loop）
```
- **优势**：无需修改生成模型架构，即插即用
- **劣势**：依赖生成模型的self-reflection能力
- **参考**：CAD-Assistant的PF1从0.747→0.979；Chat2Layout的OOB从24.8%→21.0%

**路径B：Prompt Context Injection（上下文注入模式）**

将KG查询结果作为structured context注入生成prompt：
```
[System] 你是服装版型生成助手。以下是当前设计的工艺约束：
- 面料：雪纺（弹性低，厚度0.1mm，KES�ending=0.04）
- 推荐缝型：法式缝（兼容性0.95）、窄卷边（兼容性0.90）
- 缝份规则：侧缝0.8cm、下摆1.2cm(双折边)、领口0.6cm(包边)
- Notch规则：袖窿需3个notch（肩点、前后标记各距肩点8cm）
[User] 生成一件A-line连衣裙的版型...
```
- **优势**：充分利用LLM的推理能力
- **劣势**：context window限制；LLM可能忽略注入的约束
- **适用**：ChatGarment等对话式版型生成系统

**路径C：Reward Signal for RL（强化学习奖励信号模式）**

将KG约束满足度编码为reward function：
```python
def sewing_reward(generated_pattern, fabric_type):
    violations = sckg.check_all_constraints(generated_pattern, fabric_type)
    reward = 1.0
    for v in violations:
        if v.severity == "hard":
            reward -= 1.0  # 硬约束违反：严厉惩罚
        elif v.severity == "soft":
            reward -= v.weight * 0.1  # 软约束违反：加权惩罚
    return reward
```
- **优势**：端到端优化，无需人工设计集成接口
- **劣势**：需要可微分的KG查询或近似；训练成本高
- **参考**：SVGen的GRPO训练策略使3B模型超越GPT-4o

**实验设计（三条路径对比）**：

| 指标 | 路径A (Tool) | 路径B (Prompt) | 路径C (RL) |
|------|-------------|----------------|------------|
| 缝份准确率 (vs专家标注) | 主要指标 | 主要指标 | 主要指标 |
| Notch放置F1 | 次要指标 | 次要指标 | 次要指标 |
| 缝型推荐准确率 | 次要指标 | 次要指标 | 次要指标 |
| 集成难度 (人天) | 低(3-5天) | 低(1-2天) | 高(2-4周) |
| 推理延迟增加 | +50-100ms | +0ms(预处理) | +0ms(训练时) |
| 需要修改生成模型 | 否 | 否 | 是 |

### 5. KG维护策略

**5.1 版本控制**
- KG schema采用语义版本控制（SemVer）：MAJOR.MINOR.PATCH
- 每条知识条目带有版本号、创建时间、来源、置信度
- Schema变更需要migration脚本，保证向后兼容

**5.2 自动不一致性检测**
```
定期运行:
  1. 逻辑一致性检查: ∀(A compatible_with B) ∧ (B excludes C) → ¬(A compatible_with C)
  2. 数值范围检查: seam_allowance ∈ [0.3cm, 5.0cm]（超出范围标记审核）
  3. 覆盖度检查: 每种SeamType至少有3种FabricType的参数记录
  4. 引用完整性: 每条规则至少有一个Tier 1-4来源
```

**5.3 社区贡献机制**
- 开放贡献接口：版型师/缝纫技师可提交新知识条目
- 贡献需经过至少2位审核者验证
- 贡献者获得署名和知识溯源记录
- 参考Wikidata的社区维护模式，但规模预期远小于通用KG

**5.4 范围边界控制**
- 明确范围：仅覆盖cut-and-sew工艺，不覆盖3D打印面料、超声波焊接等新兴工艺（这些作为未来扩展）
- 核心稳定区：10种基础缝型 + 5种基础面料类别 = 50个核心参数组合
- 扩展区：按需添加，每次扩展伴随验证实验

### 6. 技术实现

- **图谱存储**：轻量方案采用Property Graph（NetworkX/igraph），规模增长后迁移至Neo4j
- **查询接口**：Python API封装，输入(SeamType, PatternRegion, FabricType)三元组，输出参数+置信度
- **与Textile IR集成**：复用Textile IR的节点类型定义，SCKG节点通过`extends`关系挂载到Textile IR的PatternPiece和SeamEdge节点上
- **grammar constraint编译**：将硬约束规则编译为BNF/EBNF语法规则，供grammar-constrained decoding使用
- **序列化格式**：JSON-LD（兼容语义Web标准，便于未来与RDF生态互操作）

## 参考文献分析

### 核心参考

1. **Textile IR** (Teikari & Fuenmayor, 2026, arXiv:2601.02792)
   - 七层验证架构从语法检查到LCA评估的完整pipeline
   - 定义了PatternPiece、SeamEdge、Dart、Notch、MaterialRegion节点类型
   - SCKG定位为其Layer 6b（制造可行性）的工艺知识供给层
   - v0称其"仅为position paper无实现"系低估——其节点类型定义和验证层次设计已相当具体

2. **FALCON** (2026)
   - Grammar-constrained decoding + repair + adaptive sampling实现100%可行性
   - 证明确定性约束可直接编译进生成过程，无需外部KG查询
   - SCKG采纳此思路用于硬约束层，仅用KG处理无法grammar化的软约束

3. **CAD-Assistant** (Mallis et al., 2024)
   - Tool-augmented VLLM，constraint checker使PF1从0.747提升到0.979
   - 证明"工具调用+self-reflection"是约束满足的有效轻量方案
   - SCKG的路径A直接借鉴此模式

4. **制造知识图谱: 连接主义方法** (He & Jiang, 2019)
   - 提出ontology-based Manufacturing Knowledge Graph (MKG)，统一多因素多层次制造知识
   - 5W2H结构化查询、语义相似度计算
   - 验证了知识图谱在制造领域的实际有效性（68次引用）

5. **制造过程规划的系统化KG建模** (Wen et al., 2023)
   - 系统化的知识建模与抽取方法
   - 从工艺文档到KG的自动化pipeline设计
   - 为SCKG的LLM辅助知识抽取pipeline提供方法论参考

6. **DressCode** (He et al., SIGGRAPH 2024)
   - stitch tags编码"哪些边缝在一起"但不编码"用什么缝法"
   - 验证了工艺语义缺失的普遍性

7. **GarmentCode DSL** (Korosteleva & Sorkine-Hornung, 2023)
   - 通过程序语义保证几何有效性（100% geometric validity by construction）
   - 72%物理仿真收敛率（GarmentCodeData, 2025）
   - SCKG在GarmentCode保证的几何有效性之上，叠加制造工艺有效性

8. **SVGen** (2025)
   - 3B模型通过curriculum + CoT + GRPO超越GPT-4o
   - 证明小模型可学到精细领域知识——但SCKG的价值在于LLM不知道的长尾知识和精确数值

9. **fashioninsta.ai 行业报告**
   - "AI generates basic patterns but needs human oversight for seam allowances"
   - 直接印证SCKG存在的必要性

10. **纺织领域知识图谱构建方法** (面向工业互联网)
    - 纺织行业KG构建的已有实践
    - 为SCKG的构建提供行业特定参考

### 补充参考（v1新增）

11. **Chat2Layout** — Visual self-reflection机制，OOB从24.8%降至21.0%，证明视觉反馈循环的有效性
12. **BetterBench** — "quality trumps scale"原则，直接指导SCKG的"10种基础缝型"最小可行策略
13. **动态知识建模与融合方法** (面向定制服装生产) — 直接相关的服装定制KG工作

## 五维评分

| 维度 | v0分 | v1分 | 理由 |
|------|------|------|------|
| **新颖性** | 7 | 6 | Textile IR已有结构化节点类型定义、PLM系统隐式编码了大量工艺知识、制造KG在其他行业已有成熟实践。SCKG的新颖性在于(a)Hybrid硬+软约束架构和(b)三元上下文依赖参数解析，而非KG本身 |
| **可行性** | 6 | 6 | MVP-KG（10种缝型）可在3-6个月内由1-2位领域专家完成；但长期维护成本仍是风险。社区贡献机制和LLM辅助抽取可部分缓解 |
| **影响力** | 8 | 7 | 从KG到实际生成改进的因果链现已明确（三条路径），但路径的有效性需要实验验证。影响力取决于下游集成的实际效果而非KG本身的规模 |
| **清晰度** | 8 | 8 | Hybrid架构、三条集成路径、MVP策略、维护方案均已明确。与Textile IR的分工关系清晰 |
| **证据支撑** | 7 | 7 | 新增制造KG文献、Textile IR深度分析、FALCON/CAD-Assistant替代方案讨论。但缝制领域的KG有效性仍需实验验证 |
| **总分** | **36/50** | **34/50** | 整体下调2分，反映对相关工作认知的校正和对维护成本的诚实评估 |

## 风险与挑战

1. **知识收集的人力成本**（中等风险）：MVP-KG策略将初始规模控制在可管理范围；LLM辅助抽取降低边际成本；但专家审核仍是瓶颈
2. **知识的品牌/地区差异**（中等风险）：通过KG的多值属性和置信度标注处理——同一规则可有多个来源的不同参数值
3. **知识的隐性性**（高风险）：版型师的许多决策基于直觉和经验积累，难以通过访谈完全形式化。缓解策略：从工业DXF数据统计分析中反向推断隐性规则
4. **维护成本**（高风险）：scope控制是关键——明确不追求"全覆盖"，聚焦高频使用的核心缝型和面料组合
5. **Grammar constraint的表达力限制**（低风险）：部分约束可能难以编码为context-free grammar；对这些边界情况，回退到KG软约束查询
6. **与LLM隐式知识的重叠**（低风险）：通过聚焦长尾知识和精确数值参数，避免重复LLM已知常识
7. **下游集成效果不确定**（中等风险）：三条路径中哪条最有效需要实验验证；路径A（Tool Call）的实现成本最低，建议优先尝试
