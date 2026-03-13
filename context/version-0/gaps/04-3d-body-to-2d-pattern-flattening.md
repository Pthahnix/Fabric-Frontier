<!-- markdownlint-disable -->
# 方向四：3D体型扫描 → 2D版型展平

## 概述

从人体出发的逆向路径：通过3D体扫数据生成个性化版型。该方向的核心挑战在于如何从3D穿着状态（可能是扫描数据、照片或参数化模型）逆向推导出精确的2D版型，同时保持可制造性。代表工作包括DiffAvatar (CVPR 2024)、Inverse Garment (CGF 2024)、Dress-1-to-3 (2025)等。

---

## Gap 13: 非刚性面料的展平失真

### 问题描述

3D→2D展平算法普遍假设面料不可拉伸（isometric flattening），但现实中大量服装使用针织面料和弹性面料。弹性面料的版型设计需要"负ease"——即版型尺寸小于身体尺寸10-30%，穿着时依靠面料拉伸贴合身体。这种负ease在现有展平算法中完全未被建模。

### 现有方法的局限

| 方法 | 局限性 |
|------|--------|
| **Inverse Garment** (Yu et al., CGF 2024, arXiv:2403.06841) | 仅考虑梭织面料场景，使用可微仿真优化版型但假设面料不可拉伸 |
| **DiffAvatar** (Li et al., CVPR 2024, arXiv:2311.12194) | 在2D pattern space优化但假设developable meshes，论文明确表示"represent clothing in 2D pattern space, which ensures developable meshes and manufacturable clothing" |
| **Bang et al. (2021)** | 经典3D→2D展平方法，纯几何操作，无材料属性考虑 |
| **Sharp & Crane (2018)** | 基于曲面展平的版型逆推，同样假设面料不可拉伸 |

### 证据来源

1. **DiffAvatar论文**: "We optimize a clothing template in 2D pattern space to reproduce the captured clothing in 3D in a physical way" — 但材料模型仅包含弹性和弯曲参数，不处理大变形拉伸
2. **Inverse Garment论文**: "Built upon a differentiable cloth simulator, the optimization process is directed towards minimizing the deviation of the simulated garment shape from the target geometry" — 优化目标是形状匹配，不考虑面料拉伸导致的版型缩放
3. **真实制版实践**: 针织面料（jersey）版型通常比身体测量值小10-30%；泳装面料版型可能小40%以上。这些缩小量取决于面料弹性系数，是制版师的核心技能之一

### 影响程度

**高** — 运动服、内衣、泳装、紧身衣等弹性面料服装占全球服装市场的重要份额。不能处理弹性面料的版型生成方法在实际应用中有严重局限。

---

## Gap 14: 从穿着状态逆推静态版型的歧义性

### 问题描述

同一件衣服在不同体型、不同姿态下看起来完全不同。从单张照片逆推版型是一个严重的ill-posed问题——存在无穷多个版型可以在特定姿态下产生相似的穿着效果。特别是裙装底摆、大衣下摆等自由悬垂部分，受重力和动态影响极大，逆推版型极其困难。

### 现有方法的局限

| 方法 | 处理策略 | 剩余歧义 |
|------|----------|----------|
| **SewFormer** (Liu et al., SIGGRAPH Asia 2023) | 判别式预测，需T-pose输入 | "only works well for human images in a standard T-pose, and its performance degenerates significantly for in-the-wild images" |
| **Dress-1-to-3** (Li et al., 2025, arXiv:2502.03449) | 多视图扩散生成+可微仿真优化 | 仍需初始粗版型，优化可能陷入局部最优 |
| **DressWild** (Tao et al., 2026, arXiv:2602.16502) | VLM姿态归一化→pose-agnostic特征 | "leverages vision-language models to normalize pose variations at the image level" — 依赖VLM理解能力 |
| **NGL-Prompter** (Badalyan et al., 2026, arXiv:2602.20700) | 完全无训练，纯VLM提示 | 将问题转化为VLM理解问题，避开了几何歧义但牺牲了精度 |
| **ChatGarment** (Bian et al., CVPR 2025) | VLM输出JSON参数→GarmentCode | 受限于GarmentCode的参数空间，无法精确匹配复杂款式 |

### 证据来源

1. **SewFormer论文**: "existing garment datasets...only works well for human images in a standard T-pose"
2. **DressWild论文**: "existing feed-forward methods struggle with diverse poses and viewpoints, while optimization-based approaches are computationally expensive and difficult to scale"
3. **NGL-Prompter**: 提出training-free方案说明学习式方法在泛化上的根本困难——"VLMs struggle to reason about continuous numerical attributes"
4. **Dress-1-to-3**: "Starting with the image, our approach combines a pre-trained image-to-sewing pattern generation model for creating coarse sewing patterns" — 仍需依赖现有模型的初始预测

### 影响程度

**极高** — 这是所有"从照片重建版型"应用的核心障碍。电商、虚拟试穿、数字时尚等场景都需要从现实照片中提取版型信息。

---

## Gap 15: 体型数据的偏差

### 问题描述

SMPL/SMPL-X等参数化身体模型主要基于欧美人体数据构建。不同种族、不同年龄段、不同性别的身体比例差异未被充分建模。"Made-to-measure"（量体裁衣）应用需要更多样化的body model来覆盖全球用户。

### 现有方法的局限

- **GarmentCodeData**: 使用SMPL-based bodies生成所有训练数据
- **所有版型生成论文**: 统一使用SMPL/SMPL-X作为身体模型
- **Han (2025, IJACSA)**: 讨论了3D body scan到版型的转换，但仅考虑标准体型
- **SewingLDM** (ICCV 2025): 唯一将body shape作为生成条件的方法，但仍基于SMPL参数空间

### 证据来源

1. SMPL原始论文（Loper et al., 2015）训练数据主要来自CAESAR数据库，以欧美受试者为主
2. 服装行业实际需要的尺码系统在不同国家差异巨大（中国GB/T 1335 vs 美国ASTM vs 欧洲EN 13402）
3. 老年人、孕妇、残障人士等特殊体型在SMPL模型中表达能力有限

### 影响程度

**中高** — 对全球化应用至关重要。特别是亚洲市场（中国、日本、韩国）和特殊人群市场。

---

## Gap 16: 多尺码推板的AI化（Pattern Grading）

### 问题描述

推板（Pattern Grading）是服装制版行业的核心技术之一：从一个基础号型（base size）生成完整的多尺码系列。推板规则（grade rules）包含大量行业经验知识——不同部位的增长率不同（胸围增加4cm时，肩宽可能只增加1cm），曲线形状随尺码变化，且不同品类有完全不同的推板逻辑。目前，**没有任何AI方法能够学习或自动化推板过程**。

### 现有方法的局限

| 方法 | 与推板的关系 | 局限 |
|------|-------------|------|
| **Oh & Kim (2023)** "Automatic generation of parametric patterns from grading patterns using AI" | 唯一直接相关的论文 | 从已有推板结果**逆推**参数化版型，不是学习推板规则本身 |
| **Dress Anyone** (Chen et al., 2024, arXiv:2405.19148) | 物理仿真refitting到新体型 | 是"重新模拟"而非"推板"——不遵循工业推板的约束规则 |
| **GarmentCodeData** (Korosteleva et al., ECCV 2024) | 支持made-to-measure | 每个版型独立生成，不是从一个base size推板出系列 |

### 证据来源

1. **fashioninsta.ai报告**: 明确指出AI在grading环节仍需完全人工介入
2. **工业推板的复杂性**:
   - 等差推板（proportional grading）: 各部位按固定比例增减
   - 多点推板（multi-point grading）: 不同控制点有不同增量
   - 曲线形状变化: 领围、袖窿等曲线随尺码变化不是简单缩放
   - 品类差异: 男装、女装、童装、运动装推板规则完全不同
3. **工业软件**: Gerber AccuMark、Lectra Modaris、Optitex都有推板模块，但基于专家预设规则，非AI驱动

### 影响程度

**极高** — 这是服装工业化生产的核心环节。每个品牌、每个款式都需要推板。自动化推板将直接减少大量人工成本和时间。
