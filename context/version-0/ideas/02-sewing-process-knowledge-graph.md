<!-- markdownlint-disable -->
# Idea 02: 缝纫工艺知识图谱

## 概述

构建一个结构化的**缝纫工艺知识图谱（Sewing Construction Knowledge Graph, SCKG）**，将版型师的隐性经验知识编码为机器可查询、可推理的形式化知识。该知识图谱以构造操作（缝合类型、折边工艺、闭合方式等）为节点，以约束关系（材料兼容性、工艺序列、工具要求等）为边，以工艺参数（缝份宽度、标记点位置、grain line方向等）为属性。通过将SCKG作为约束层集成到AI版型生成模型中，实现从"几何版型"到"可制造版型"的跨越。

## 指向的Gap

- **Gap 3: 缝纫语义的缺失** — 现有方法仅编码"哪些边缝在一起"，不区分缝合工艺类型（平缝、包缝、暗缝等）
- **Gap 17: 缝份（Seam Allowance）的自动生成** — 缝份宽度因部位、工艺、面料而异，没有AI系统整合这些规则
- **Gap 18: 标记点（Notch）和对位点的缺失** — 工业版型需要notch、grain line、dart point等标记
- **Gap 19: 面料约束的缺失** — 面料特性直接决定多个版型参数

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察Gap 17-19的共性**：缝份、标记点、面料约束这三个Gap的共同根源是——AI版型生成方法**缺乏服装制造工艺知识**。这些知识存在于经验丰富的版型师大脑中，但从未被系统化地形式化编码。
2. **知识图谱的适配性**：这些工艺知识天然适合知识图谱表示——实体（缝合类型、面料类型、版型部位）之间存在丰富的关系（适用于、兼容、要求、排斥），且这些关系具有明确的规则性。
3. **集成策略**：知识图谱可以作为AI生成模型的**后处理约束层**（验证并修正生成结果）或**条件输入**（在生成过程中施加工艺约束）。

### 与现有工作的关系

- **Textile IR** (Teikari & Fuenmayor, 2026): 提出中间表征概念，但未涉及工艺知识的形式化
- **ASTM D6673标准**: 定义了数字版型数据格式，但不包含工艺规则
- **fashioninsta.ai报告**: 明确指出AI在缝份和推板环节仍需人工介入——正是因为缺乏工艺知识

## 详细设计

### 1. 知识图谱本体设计

**实体类型（Entity Types）**：

| 实体类型 | 示例 | 属性 |
|----------|------|------|
| 缝合类型（SeamType） | 平缝、包缝、暗缝、法式缝、flat-felled seam | 适用面料、所需设备、缝份需求、强度等级 |
| 折边工艺（HemFinish） | 单折边、双折边、窄卷边、包边、粘合边 | 折边宽度、适用位置、面料要求 |
| 闭合方式（ClosureType） | 拉链、纽扣、暗扣、魔术贴、绳扣 | 安装位置、缝份需求、面料兼容性 |
| 面料类型（FabricType） | 棉布、丝绸、牛仔、针织、雪纺 | 弹性、重量、厚度、grain方向性 |
| 版型部位（PatternRegion） | 领口、肩缝、侧缝、下摆、袖窿、腰线 | 曲率、受力方向、美观要求 |
| 标记类型（MarkType） | V-notch、T-notch、drill hole、grain line、dart point | 形状、大小、适用位置 |

**关系类型（Relation Types）**：

| 关系 | 示例 | 语义 |
|------|------|------|
| requires_seam_allowance(SeamType, PatternRegion) → width | 平缝+侧缝 → 1.0cm | 特定缝合+部位的缝份规则 |
| compatible_with(SeamType, FabricType) → bool | 法式缝+薄面料 → true | 工艺-面料兼容性 |
| notch_placement(SeamType, PatternRegion) → positions | 袖窿缝合 → [肩点, 前后标记] | 标记点放置规则 |
| grain_direction(PatternRegion, FabricType) → angle | 裙片+格纹面料 → 0°(经向) | grain line方向规则 |
| follows(Operation, Operation) → sequence | 缝合侧缝 → 折下摆边 | 工艺序列约束 |

### 2. 知识来源与构建方法

**a) 专家知识提取**
- 访谈版型师和缝纫技师，结构化记录工艺决策规则
- 分析服装制版教科书（如Aldrich的《Metric Pattern Cutting》、Armstrong的《Patternmaking for Fashion Design》）

**b) 标准文档解析**
- ASTM D6673（数字版型标准）
- ISO 4916（缝合类型标准）
- 各品牌的版型规范文档

**c) 数据驱动发现**
- 从工业版型数据（DXF文件）中自动提取缝份宽度分布、标记点模式
- 统计不同部位、面料、缝合类型的参数分布

### 3. 集成到AI版型生成

**模式A：后处理约束层**
```
AI生成净版型（panel geometry + stitch tags）
    ↓
SCKG查询：每条stitch edge的缝合类型 → 缝份宽度
    ↓
自动添加缝份偏移（offset curve）
    ↓
SCKG查询：每条缝合线的notch规则 → 标记点位置
    ↓
添加标记点
    ↓
输出可制造版型
```

**模式B：生成时约束注入**
- 将SCKG中的约束编码为loss函数或条件向量
- 在GarmentDiffusion/SewingGPT的生成过程中施加约束
- 例如：惩罚生成的panel宽度超过面料幅宽

### 4. 技术实现

- **图谱存储**：Neo4j或RDF三元组存储
- **查询接口**：SPARQL或Cypher查询语言
- **规则引擎**：基于图谱的推理引擎，自动推导未显式编码的规则
- **与版型生成模型的接口**：Python API，输入panel geometry+stitch info，输出augmented pattern

## 参考文献分析

### 核心参考

1. **fashioninsta.ai 行业报告**
   - "AI generates basic patterns but needs human oversight for seam allowances" — 直接说明了工艺知识缺失的问题
   - "46% fabric reduction possible" — 暗示如果AI能整合面料约束知识，将有巨大的效率提升

2. **ASTM D6673 标准**
   - 定义了数字版型数据交换的标准格式
   - 包含panel geometry、seam connections、grading data等字段
   - 本提案的知识图谱可以视为ASTM D6673的"语义扩展"——不仅存储数据格式，还存储工艺规则

3. **Textile IR** (Teikari & Fuenmayor, 2026, arXiv:2601.02792)
   - 提出双向中间表征概念，旨在统一学术和工业格式
   - 仅为position paper无实现
   - 本提案与Textile IR互补——Textile IR解决格式转换，SCKG解决工艺知识

4. **DressCode论文** (He et al., SIGGRAPH 2024)
   - stitch tags编码"哪些边缝在一起"但不编码"用什么缝法"
   - 每条边的stitch tag是3D坐标中点，不含工艺信息

5. **所有核心论文的输出格式**
   - 均为panel顶点坐标 + edge参数 + stitch tags，无缝份、无标记点、无工艺信息
   - 验证了本提案的必要性

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 7/10 | 知识图谱在NLP/推荐系统中成熟，但应用于版型工艺领域是新的 |
| **可行性** | 6/10 | 知识图谱的构建技术成熟，主要挑战在于工艺知识的收集和形式化——需要领域专家的深度参与 |
| **影响力** | 8/10 | 一旦构建完成，可以作为通用基础设施被所有AI版型生成方法复用，直接桥接学术-工业鸿沟 |
| **清晰度** | 8/10 | 本体设计和集成策略明确，实施路径清晰 |
| **证据支撑** | 7/10 | 大量证据表明工艺知识缺失是核心问题；知识图谱在其他领域的成功案例提供了方法论信心 |
| **总分** | **36/50** | |

## 风险与挑战

1. **知识收集的人力成本**：需要大量领域专家参与，知识提取过程漫长
2. **知识的品牌/地区差异**：不同品牌、不同国家的工艺标准不同，需要处理知识的多样性和冲突
3. **知识的隐性性**：许多工艺决策是版型师基于直觉做出的，难以形式化
4. **维护成本**：时尚工艺在不断演变，知识图谱需要持续更新
5. **集成难度**：将离散的图谱知识与连续的神经网络生成过程结合，需要创新的接口设计
