<!-- markdownlint-disable -->
# 方向三：生成范式的比较与统一

## 概述

AI版型生成领域目前存在四种根本不同的生成范式，各自采用截然不同的版型表示、模型架构和生成策略：自回归序列生成（DressCode/SewingGPT）、扩散模型并行去噪（GarmentDiffusion）、程序合成（Design2GarmentCode/ChatGarment）、以及图像化表征（GarmageNet）。然而，这些范式之间缺乏公平的对比实验，每篇论文使用不同的数据集、指标和评估协议，使得领域进展难以衡量。此外，各范式都面临序列长度/可扩展性、拓扑变化处理、以及编辑一致性等共性挑战。

---

## Gap 9: 自回归 vs 扩散 vs 程序合成——范式间缺乏公平对比

### 问题描述

当前存在四种根本不同的版型生成范式，但**没有任何标准化基准**能实现公平比较：

1. **自回归序列生成（Autoregressive）**
   - 代表：SewingGPT/DressCode (He et al., SIGGRAPH 2024)
   - 方法：将版型参数量化为离散token，组织为1D序列，使用GPT-2架构自回归生成
   - 表示：版型 → token序列（panel类型 + edge参数 + stitch tags），总长约1786 tokens

2. **扩散模型并行去噪（Diffusion）**
   - 代表：GarmentDiffusion (Li et al., IJCAI 2025)
   - 方法：将版型表示为edge token矩阵，使用Diffusion Transformer (DiT)并行去噪所有edge tokens
   - 表示：固定大小矩阵（max 16 panels × 14 edges），每个edge token含起点、控制点、弧参数、stitch信息
   - 声称比SewingGPT快100倍，序列长度短10倍

3. **程序合成（Program Synthesis）**
   - 代表：Design2GarmentCode (Zhou et al., CVPR 2025), ChatGarment (Bian et al., CVPR 2025)
   - 方法：LLM/VLM生成GarmentCode DSL程序或JSON参数
   - 表示：可执行程序代码（Python）或结构化JSON

4. **图像化表征（Image-like Representation）**
   - 代表：GarmageNet (SIGGRAPH Asia 2025)
   - 方法：将版型光栅化为2D"garmage"图像，利用成熟的图像生成模型
   - 表示：多通道2D图像编码几何和拓扑信息

### 现有评估的不公平性

| 论文 | 使用的数据集 | 使用的指标 | 比较对象 |
|------|------------|-----------|----------|
| SewingGPT/DressCode (SIGGRAPH 2024) | 自建20.3K（GarmentCode生成） | Panel L2, #Panel acc, #Edge acc, Rot L2, Trans L2, Stitch F1 | NeuralTailor, SewFormer |
| GarmentDiffusion (IJCAI 2025) | DressCode数据集（重新标注edge tokens） | 同上指标（自行复现SewingGPT baseline） | SewingGPT（自行复现） |
| AIpparel (CVPR 2025) | 自建GCD-MM 120K | CLIP score, FID, garment-specific metrics | 无直接版型对比 |
| ChatGarment (CVPR 2025) | GarmentCodeRC生成 | 分类准确率, user study | Design2GarmentCode |
| Design2GarmentCode (CVPR 2025) | GarmentCodeData | 渲染相似度, user study | ChatGarment |
| GarmageNet (SIGGRAPH Asia 2025) | 自建garmage数据集 | 图像质量指标 + 版型准确率 | SewFormer |

**关键问题**：
- GarmentDiffusion声称优于SewingGPT，但使用的是自行复现的SewingGPT baseline，而非原作者代码
- 各方法使用不同的train/test split，甚至不同的底层数据
- 程序合成方法（Design2GarmentCode, ChatGarment）使用user study而非自动指标，与序列/扩散方法的评估不可比
- 没有任何论文在**同一数据集、同一指标、同一评估协议**下比较所有范式

### 证据来源

1. **GarmentDiffusion论文**: 比较表中SewingGPT的数值来自作者自行复现，而非原始论文报告值，存在复现偏差风险
2. **DressCode论文**: 仅与NeuralTailor和SewFormer比较，这两者都是较早的方法，未与同期的程序合成方法对比
3. **AIpparel论文**: 使用CLIP score和FID等图像级指标，与其他方法的几何级指标（Panel L2等）不可直接比较
4. **领域现状**: 每个研究组都在"自己的擂台上"展示优势，没有公共的竞技场

### 影响程度

**极高（Critical）** — 缺乏公平比较使得领域无法确定哪种范式真正优越，研究者无法做出有根据的技术选型决策。这阻碍了资源的有效配置和技术的收敛。类比NLP领域：GLUE/SuperGLUE基准的出现极大推动了模型架构的迭代和优化。版型生成领域急需类似的统一基准。

---

## Gap 10: 序列长度与可扩展性瓶颈

### 问题描述

自回归和扩散方法都面临**固定容量限制**，当服装复杂度增加时（更多版片、更复杂的曲线、更多缝合关系），现有表示可能不足以编码所有信息。

### 各范式的容量限制

| 范式 | 表示容量 | 限制来源 | 对复杂服装的影响 |
|------|---------|---------|----------------|
| **SewingGPT** | 1786 tokens | GPT-2 context window (1024)，实际使用约1786 | 一件复杂夹克（20+ panels）可能超出token预算 |
| **GarmentDiffusion** | 16 panels × 14 edges = 224 edge tokens | 固定矩阵大小，不足部分用padding | 超过16个panel的服装无法表示 |
| **程序合成** | 理论上无限（程序可任意长） | 受限于LLM context window和代码生成能力 | 最灵活，但生成质量随复杂度下降 |
| **GarmageNet** | 固定图像分辨率 | 光栅化精度受分辨率限制 | 细节可能丢失 |

### 具体证据

1. **DressCode/SewingGPT**: "We quantize each attribute...resulting in a flat 1D token sequence" — 将所有panel的所有edge参数展平为1D序列。一件简单连衣裙可能有6-8个panel，一件西装上衣有12-15个panel，一件大衣外加衬里可能有30+个panel。当前的token预算显然不够。

2. **GarmentDiffusion**: 固定16 panels × 14 edges的矩阵表示。论文使用padding处理panel数少于16的情况，但没有讨论panel数超过16的场景。实际上GarmentCodeData中的复杂服装可能包含20+个panel。

3. **对比参照——LLM的context扩展**: NLP领域通过RoPE、ALiBi、sliding window等技术将context从2K扩展到128K+。版型生成领域**完全没有**类似的序列长度扩展工作。

### 影响程度

**高（High）** — 限制了可生成服装的复杂度。简单T恤和裙子可以处理，但工业级的夹克、大衣、套装等复杂服装超出了当前方法的表示能力。这将AI版型生成的实际应用范围严重限制在简单款式上。

---

## Gap 11: 拓扑变化的处理——添加/删除版片

### 问题描述

所有当前方法都假设**固定的版型拓扑结构**（固定数量的panel和固定的缝合连接模式），无法在生成过程中动态添加或删除版片。这对以下场景至关重要：

- **添加口袋**: 需要新增panel（口袋布、口袋盖、贴袋面等），并建立与主体panel的缝合关系
- **添加领子/袖口**: 新增独立的panel，具有特殊的缝合拓扑
- **复杂多片式服装**: 西装上衣的公主线版型将前片分为3片，每片有不同的grain line和缝合方式
- **设计变体**: 同一基础版型的不同变体可能有不同数量的panel（如A字裙 vs 八片裙）

### 现有方法的局限

1. **GarmentDiffusion**: 使用固定大小的矩阵表示（16 panels × 14 edges），panel数量在推理前就已固定。虽然可以通过padding mask标记"空"panel，但生成时无法动态决定"需要多少个panel"。

2. **SewingGPT/DressCode**: 虽然自回归生成理论上可以生成变长序列，但训练数据的panel数量分布有限，且序列格式假设了特定的panel组织结构。

3. **AIpparel**: 在limitations中明确声明："constrained to garments representable by manifold surfaces. Design elements like pockets require non-manifold structures." — 口袋等需要非流形结构的设计元素**根本无法表示**。

4. **ChatGarment**: 受限于GarmentCode的组件库。如果GarmentCode没有定义某种panel类型，ChatGarment就无法生成它。论文中的GarmentCodeRC扩展增加了一些新组件，但仍是手工预定义的有限集合。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| AIpparel (CVPR 2025) | §Limitations: "Design elements like pockets require non-manifold structures" |
| GarmentDiffusion (IJCAI 2025) | 固定16 panel矩阵，padding处理变长 |
| 工业实际 | 一件男式衬衫有15-20个panel（含口袋、领子、袖口、门襟），一件西装上衣有25-35个panel |
| GarmentCode DSL | 预定义组件库，新panel类型需手工编程添加 |

### 影响程度

**高（High）** — 拓扑固定意味着AI只能生成"模板内"的服装。真正的设计创新往往涉及版片结构的根本改变（如解构主义设计、不对称裁剪、创新接缝位置），这些都需要动态的拓扑变化能力。

---

## Gap 12: 编辑一致性——局部修改导致全局破坏

### 问题描述

在交互式版型设计中，设计师常需对已生成的版型进行**局部编辑**（如加长袖子、收窄腰围、改变领型），同时期望其他部分保持不变。然而，当前方法普遍无法保证编辑的局部性——修改一个参数可能导致其他部分发生意外变化。

### 现有方法的编辑能力分析

| 方法 | 编辑能力 | 一致性问题 |
|------|---------|-----------|
| **ChatGarment** (CVPR 2025) | 支持多轮对话编辑 | 论文结论明确承认："altering the skirt length might unintentionally affect other parts of the garment"（改变裙长可能无意中影响服装其他部分） |
| **Design2GarmentCode** (CVPR 2025) | 通过修改程序参数实现 | 参数间存在耦合——修改一个参数可能触发程序中其他计算路径的变化 |
| **DressCode/SewingGPT** (SIGGRAPH 2024) | 无编辑接口 | 只能重新生成整个版型 |
| **GarmentDiffusion** (IJCAI 2025) | 无编辑接口 | 只能重新生成整个版型 |
| **AIpparel** (CVPR 2025) | 支持editing token | 编辑效果受限于tokenization粒度 |

### 编辑一致性问题的根源

1. **程序合成方法的参数耦合**: GarmentCode程序中，版型的各个部分通过共享参数和约束相互关联。例如，领口深度影响前片形状，进而影响肩线位置，最终影响袖片形状。这种级联效应使得纯参数修改难以保持局部性。

2. **序列/扩散方法的全局重生成**: SewingGPT和GarmentDiffusion没有"部分编辑"的机制。任何修改都需要重新生成整个版型，无法保证与原版型的一致性。理论上可以通过inpainting（局部重生成）实现，但版型的stitch约束（哪些边缝合在一起）使得局部重生成极易破坏全局拓扑一致性。

3. **对比参照——图像编辑领域**: 图像编辑中的InstructPix2Pix、SDEdit等方法已经展示了保持未编辑区域一致性的能力。但版型编辑比图像编辑**更难**，因为版型有严格的几何和拓扑约束（边长匹配、角度连续、stitch一致），而图像只需视觉上看起来合理。

### 证据来源

1. **ChatGarment论文**: §Conclusion明确指出局部编辑导致全局变化的问题，将其列为核心局限
2. **Design2GarmentCode论文**: 虽然展示了参数编辑案例（Figure中的parameter slider），但未量化评估编辑一致性
3. **实际制版工作流**: 专业制版师在修改版型时会遵循严格的规则（如修改胸围同时必须调整侧缝、肩宽、袖窿曲线的对应量），确保修改的系统性一致性。AI方法缺乏这种约束传播机制

### 影响程度

**高（High）** — 交互式编辑是版型设计的核心工作流。如果每次微小修改都可能破坏整体版型的一致性，设计师将无法信任AI工具，只能将AI生成的版型作为"起点"再完全手工调整，大大降低了AI辅助设计的实际价值。
