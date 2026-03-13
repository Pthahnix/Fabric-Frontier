<!-- markdownlint-disable -->
# 方向三：生成范式的比较与统一

## v0→v1 变更说明

| 变更项 | 说明 |
|--------|------|
| **Gap 9 重构** | 接受 Rebuttal 对 GLUE 类比的批评（BetterBench、SAIBench 证据有力），但保留"缺乏可比性"作为核心问题。从"构建统一基准"转向"设计标准化评估协议+共享测试集"，降级为 High（原 Critical）。新增第五范式 GarmentGPT（ICLR 2026）的 RVQ-VAE latent tokenization |
| **Gap 10 降级并重构** | 接受 Rebuttal 关于 context window 扩展和层次化表示的论点。将序列长度问题从"独立 Gap"降级为 Gap 9 内讨论的范式特性之一，但保留其作为自回归范式结构性弱点的分析。融入 GarmentDiffusion 在 GarmentCodeData 上实证的 18K token 不可行性证据和 SDAR 范式融合新进展 |
| **Gap 11 重新聚焦** | 接受 Rebuttal 关于程序合成和 DETR 式集合预测的反驳。但基于 GarmageNet 的 N=32 固定 padding、GarmentDiffusion 的 M×N 矩阵、以及 AIpparel 明确承认的非流形限制，将问题重新定义为"工业级拓扑复杂度"而非"是否可变"。降级为 Medium |
| **Gap 12 合并+替换** | 接受 Rebuttal 关于与 Gap 8 重叠的批评，将编辑一致性内容合并至方向二。替换为新 Gap 12：范式融合的理论基础与实践路径，基于 SDAR、GarmentGPT 等 2025-2026 年新证据 |
| **新增范式** | 新增 GarmentGPT（RVQ-VAE latent tokenization + VLM autoregressive，ICLR 2026）作为第五种范式进入分析框架 |

---

## 概述

AI 版型生成领域目前存在五种根本不同的生成范式，各自采用截然不同的版型表示、模型架构和生成策略：

1. **自回归序列生成**（DressCode/SewingGPT, SIGGRAPH 2024）—— 将版型参数 vector-quantize 为离散 token 的 1D 序列，GPT-2 架构自回归生成
2. **扩散模型并行去噪**（GarmentDiffusion, IJCAI 2025）—— edge token 矩阵表示，DiT 并行去噪
3. **程序合成**（Design2GarmentCode / ChatGarment, CVPR 2025）—— LLM/VLM 生成 GarmentCode DSL 程序或 JSON 参数
4. **图像化表征**（GarmageNet, SIGGRAPH Asia 2025）—— 将版型光栅化为 per-panel geometry image，latent diffusion 生成
5. **离散潜空间 tokenization**（GarmentGPT, ICLR 2026）—— RVQ-VAE 将连续曲线 tokenize 为 codebook 索引，VLM 自回归预测离散 token 序列

这些范式之间缺乏标准化的可比评估，每篇论文使用不同的数据集分割、指标定义和评估协议。此外，各范式在序列容量/可扩展性、拓扑复杂度处理等维度展现出截然不同的结构性特征，需要系统性的理解而非简单的优劣排名。

---

## Gap 9: 跨范式评估的不可比性——缺乏标准化评估协议

### 问题描述

五种版型生成范式使用根本不同的输入/输出表示和评估框架，现有论文间的性能比较存在系统性偏差，使得领域无法做出有根据的技术选型判断。

**核心问题不是"缺乏统一基准"，而是"缺乏最低限度的评估可比性"。**

### 现有评估的不可比性证据

| 论文 | 数据集 | 关键指标 | 比较对象 | 可比性问题 |
|------|--------|---------|----------|-----------|
| DressCode (SIGGRAPH 2024) | 自建 20.3K (GarmentCode) | Panel L2, #Panel acc, #Edge acc, Stitch F1 | NeuralTailor, SewFormer | 仅与早期方法对比 |
| GarmentDiffusion (IJCAI 2025) | DressCode + GarmentCodeData | 同上指标 | SewingGPT（自行复现） | SewingGPT 在 GarmentCodeData 上因 18K token 长度无法训练全集，仅用 ≤10 panels、≤12 edges 子集 |
| ChatGarment (CVPR 2025) | GarmentCodeRC | 分类准确率, user study | Design2GarmentCode | 非自动指标，不可与几何指标直接比较 |
| Design2GarmentCode (CVPR 2025) | GarmentCodeData | 渲染相似度, user study | ChatGarment | 同上 |
| GarmageNet (SIGGRAPH Asia 2025) | GarmageSet 14,801 | 版型准确率 + 3D garment quality | SewFormer | 使用自建工业级数据集，与合成数据方法不可直接比 |
| GarmentGPT (ICLR 2026) | GarmentCodeData + Real-Garments Benchmark | Panel Acc 95.62%, Stitch Acc 81.84% | 现有方法 | 首次建立 real-garment benchmark，但其 RVQ-VAE 表示与其他方法的原始坐标表示在指标语义上不同 |
| AIpparel (CVPR 2025) | 自建 GCD-MM 120K | CLIP score, FID | 无直接版型对比 | 图像级指标与几何级指标不可比 |

**关键不公平性实例**：

1. **GarmentDiffusion vs SewingGPT**：GarmentDiffusion 声称优于 SewingGPT，但 SewingGPT 在 GarmentCodeData 全集上因序列长度（>18K tokens）根本无法训练，仅在 ≤10 panels × ≤12 edges 的子集上评估。这不是同等条件的比较，而是在对手的弱项上进行的选择性测试。
2. **GarmageNet vs 其他方法**：GarmageNet 使用 GarmageSet（14,801 工业级版型），而其他方法使用 GarmentCodeData（程序合成数据）。数据分布的根本差异使得跨论文的数值比较失去意义。
3. **程序合成方法的评估孤岛**：Design2GarmentCode 和 ChatGarment 主要依赖 user study 和渲染相似度，与序列/扩散方法使用的几何精度指标（Panel L2、Stitch F1）体系完全不同。

### 现有方法的局限

**v0 版本呼吁构建类似 GLUE/SuperGLUE 的统一基准，但这一方向受到了有力质疑**：

1. **BetterBench (Stanford, NeurIPS 2024)**：对 24 个主流 AI benchmark 进行了系统性元评估。MMLU——最接近 GLUE 范式的单一基准——在 46 项质量标准评估中排名偏低。更重要的是，研究发现大多数 benchmark 上的模型排名差异缺乏统计显著性。
2. **SAIBench**：明确倡导模块化可组合评估系统，将问题定义、模型接口和评估指标三个维度解耦，反对构建单体基准（monolithic benchmark）。
3. **"Can We Trust AI Benchmarks?" (2025)**：指出 benchmark 是规范性工具（normative instruments），单一基准的主导地位会创造 path dependencies 和 scientific monoculture。

**然而，这些批评针对的是"构建统一排行榜"这一目标，并不否定"标准化评估协议"的必要性。** 当前的问题不是需要一个分出高下的排行榜，而是连最基本的评估可比性都不存在——每种方法都在"自己的擂台上"展示优势。

### 修正后的研究方向

基于 BetterBench 和 SAIBench 的洞察，应采用以下策略而非构建单体基准：

1. **标准化最低评估协议**：定义每种范式必须报告的最低指标集合（共享的最终几何精度指标），同时允许各范式报告范式特定的中间质量指标
2. **共享测试用例库**：提供一组覆盖简单到复杂的标准服装规格（如：6-panel 连衣裙、15-panel 衬衫、25-panel 西装），所有方法在同一组规格上评估
3. **模块化评估工具包**：参照 SAIBench 的三维度解耦设计（问题/接口/指标），提供可组合的评估组件
4. **统一的 end-to-end 可缝制性测试**：所有范式最终输出都需要通过物理仿真验证，这是唯一真正公平的端到端评估点

### 证据来源

1. GarmentDiffusion 论文 Table 2：SewingGPT 在 GarmentCodeData 子集上的受限评估
2. GarmentDiffusion 论文 §4.1："SewingGPT cannot be trained on the entire GarmentCodeData due to its long context length"
3. GarmageNet 论文 §6：GarmageSet 使用工业级版型数据，与 GarmentCodeData 的程序合成数据在分布上根本不同
4. GarmentGPT (ICLR 2026)：建立 Real-Garments Benchmark，但其 RVQ-VAE 离散表示引入了新的评估维度差异
5. BetterBench (Stanford, NeurIPS 2024)：AI benchmark 元评估框架
6. SAIBench：模块化可组合评估系统设计

### 影响程度评估

**高（High）**（v0 为 Critical，降级原因：问题从"需要统一基准"转为"需要标准化协议"，后者在工程上更可行且已有参考方案）

缺乏评估可比性使得领域无法判断不同范式在不同任务维度上的实际优劣势。但这是一个可以通过社区协调逐步解决的问题（标准化协议 + 共享测试集），而非需要从零构建统一基准的基础性研究问题。

---

## Gap 10: 自回归范式的结构性容量瓶颈

### 问题描述

自回归序列生成范式在处理复杂版型时面临**结构性的序列长度瓶颈**，这一瓶颈已在 GarmentDiffusion 的实证中被确认：SewingGPT 在 GarmentCodeData（最大 37 panels × 39 edges）上的 token 序列长度超过 18,000，导致训练和推理在实践中不可行。

**v0 版本将此问题描述为一个独立的、影响所有范式的 Gap。v1 版本接受 Rebuttal 的部分论点，将其重新定位为自回归范式特有的结构性弱点，同时讨论各范式的容量扩展路径。**

### 各范式的容量限制与扩展路径

| 范式 | 当前容量 | 限制本质 | 扩展路径 | 扩展可行性 |
|------|---------|---------|---------|-----------|
| **SewingGPT** | ~1,786 tokens (DressCodeData); >18K (GarmentCodeData 全集不可训练) | Token 序列长度与版片数×边数线性增长 | Context window 扩展; 层次化 tokenization (CAD-LLaMA SPCC) | 中期可行（1-2年） |
| **GarmentGPT** | RVQ-VAE codebook 索引序列 | 受 VLM context window 和 codebook 容量限制 | Residual VQ 多级量化; 更大 codebook | 中期可行 |
| **GarmentDiffusion** | M×N 固定矩阵 (GarmentCodeData: 37×39=1,443 tokens) | 需预设最大 panel/edge 数; padding 浪费 | 动态矩阵大小; sparse attention | 短期可行 |
| **GarmageNet** | N=32 panels, 256×256 分辨率/panel | 固定 panel 数和图像分辨率 | 增加 N; 超分辨率 | 短期可行 |
| **程序合成** | 理论无限（程序可复用函数） | LLM context window; 生成质量随复杂度对数下降 | 更强 LLM; 模块化程序生成 | 最灵活 |

### 具体证据

1. **GarmentDiffusion 论文 §4.1 实证确认**："SewingGPT cannot be trained on the entire GarmentCodeData due to its long context length. To address this problem, we created a subset of GarmentCodeData by filtering out those patterns with #edges/panel > 12 and #panels/pattern > 10。" 这是对 v0 Gap 10 的直接实证支持——自回归方法在最大公开数据集上已经遇到了不可逾越的序列长度障碍。

2. **CAD-LLaMA (2025) 的 SPCC 层次化表示**：将 CAD 命令组织为结构化层级后，命令准确率从 39.17% 提升至 80.41%。这证明了更聪明的 tokenization 策略可以有效缓解序列长度问题，但尚未在版型生成领域得到验证。

3. **SDAR (2025) 的范式融合证据**：SDAR 将自回归和扩散范式融合为 block-wise diffusion + global autoregression，在保持 AR 级别性能的同时实现并行解码。30B MoE 模型在科学推理 benchmark（GPQA, ChemBench）上超越纯 AR 对应模型。这表明 AR 和扩散不必是互斥选择。

4. **GarmentGPT (ICLR 2026) 的折中方案**：通过 RVQ-VAE 将连续曲线 tokenize 为 codebook 索引，在保持 VLM 自回归架构的同时显著压缩序列长度。Panel Accuracy 95.62%、Stitch Accuracy 81.84% 表明离散潜空间 tokenization 是一条有效路径。

### Rebuttal 论点的评估

Rebuttal 提出三个反驳：(1) context window 指数扩展将使问题自然过时；(2) 层次化表示有效缩短序列；(3) 其他范式天然不受此限制。

**接受程度**：

- **论点(1)部分接受**：Context window 确实在快速扩展（2年内从 8K→2M），但版型生成的问题不仅是 context window 大小，还包括长序列中的注意力稀释、训练效率下降等结构性问题。即使 context window 足够，18K+ token 序列的训练 FLOPs 消耗仍然是实际障碍。
- **论点(2)完全接受**：CAD-LLaMA 和 GarmentGPT 的 RVQ-VAE 都证明了更好的 tokenization 可以大幅缓解问题。
- **论点(3)完全接受**：这是降低本 Gap 严重性的核心原因——序列长度确实是自回归特有的问题，选择其他范式可以规避。

### 影响程度评估

**中（Medium）**（v0 为 High，降级原因：(1) 其他范式不受此限制提供了替代路径；(2) GarmentGPT 的 RVQ-VAE 证明了在 AR 框架内也有缓解方案；(3) 问题有明确的技术解决路径，时间维度上 1-2 年内大概率显著缓解）

此 Gap 对选择自回归范式的研究者仍然重要——它定义了该范式的结构性边界，并指向 latent tokenization 和层次化表示作为关键改进方向。

---

## Gap 11: 工业级拓扑复杂度的处理能力

### 问题描述

v0 版本将此问题定义为"所有方法假设固定拓扑，无法添加/删除版片"。Rebuttal 正确指出程序合成范式天然支持可变拓扑（条件分支），且 DETR 式集合预测架构可处理可变数量输出。

**v1 重新定义问题**：核心挑战不是"能否生成不同数量的 panel"（多种范式已有方案），而是**工业级的拓扑复杂度**——包括非流形结构（口袋、内衬）、多层构造、复杂缝合图（多对多、部分边映射）——远超当前所有方法的处理能力。

### 各范式的拓扑处理能力现状

| 范式 | 可变 panel 数 | 非流形结构 | 多层构造 | 复杂缝合图 |
|------|-------------|-----------|---------|-----------|
| **SewingGPT** | 理论可行（变长序列） | ✗ | ✗ | 仅 1-to-1 全边映射 |
| **GarmentDiffusion** | 通过 padding mask（≤37 panels） | ✗ | ✗ | 仅 1-to-1 全边映射 |
| **程序合成** | ✓（条件分支） | 受限于 GarmentCode 组件库 | 受限于 DSL 定义 | 受限于 DSL 支持 |
| **GarmageNet** | 通过 padding（≤32 panels） | 明确承认为 limitation | 明确承认为 limitation | 支持 point-to-point，优于 edge-based |
| **GarmentGPT** | 通过 compositional generation | 未报告 | 未报告 | 通过 VLM 推理 |

### 具体证据

1. **AIpparel (CVPR 2025) §Limitations**："Design elements like pockets require non-manifold structures" —— 口袋等工业基础设计元素根本无法表示。

2. **GarmageNet 论文 §10 Limitations**：明确列出三项拓扑限制：
   - "Handling of interior seams and pocket-like components"
   - "Multi-layer designs and small-panel handling"
   - 暗示 N=32 的固定 panel 数上限是工程选择而非理论限制

3. **GarmageNet §2.1.3 对 GarmentCodeData 的批评**：指出 GarmentCodeData "substantially simplifies realities of production data"——工业版型包含 multi-edge-loop panels、复杂轮廓（如螺旋线），需要 many-to-many partial-edge mappings 以支持多层和密集褶皱，而 GarmentCodeData 仅编码 one-to-one full-edge correspondences。

4. **工业实际**：一件男式衬衫有 15-20 个 panel（含口袋、领子、袖口、门襟），一件西装上衣有 25-35 个 panel。更复杂的是，这些 panel 之间的缝合关系不是简单的边对边映射，而是涉及 ease 分配、聚褶、裥位对应等复杂工艺约束。

### Rebuttal 论点的评估

Rebuttal 提出四个反驳：(1) GarmentCode 条件分支支持可变拓扑；(2) DETR 式集合预测可处理可变数量输出；(3) 这是架构选择和数据问题而非基础限制；(4) 可解耦为拓扑预测+几何生成两阶段。

**接受程度**：

- **论点(1)部分接受**：程序合成确实支持可变拓扑，但受限于 GarmentCode 预定义的组件库。GarmageNet 论文对 GarmentCodeData 的批评直接说明了这一局限——程序合成生成的版型"substantially simplifies realities of production data"。
- **论点(2)接受**：DETR 式架构在理论上可行，GarmentDiffusion 和 GarmageNet 的 padding + mask 机制已是朝此方向的初步尝试。
- **论点(3)部分接受**：可变 panel 数确实是工程问题。但非流形结构（口袋、内衬）和复杂缝合图（many-to-many partial-edge）需要**表示层面的根本创新**，不仅仅是架构选择——所有当前的 edge-based stitch 表示都无法编码这些工业必需的拓扑特征。GarmageNet 的 point-to-point stitch 表示是一个有意义的进步，但仍需验证其在复杂拓扑下的可靠性。
- **论点(4)接受**：两阶段解耦（先拓扑、再几何）是合理的工程路径。

### 影响程度评估

**中（Medium）**（v0 为 High，降级原因：(1) "可变 panel 数"已有多种技术方案；(2) 真正的难题——非流形结构和复杂缝合图——是工业化部署问题而非学术核心问题，当前学术系统主要服务于简单到中等复杂度的版型生成场景）

此 Gap 对学术研究的影响有限（多数论文的目标场景不需要工业级拓扑复杂度），但对实际产业应用是关键瓶颈。GarmageNet 的工业级数据集（GarmageSet）和 point-to-point stitch 表示标志着向解决此问题迈出的重要一步。

---

## Gap 12: 范式融合的理论基础与实践路径（新增）

### 问题描述

五种版型生成范式目前被视为互斥的替代方案，但 2025-2026 年的新进展表明，范式融合可能产生超越任何单一范式的系统。然而，**如何有原则地融合不同范式，使每种范式处理其最擅长的子问题**，目前缺乏理论框架和系统性的实践探索。

### 范式融合的新证据

1. **SDAR (2025)**：成功将自回归和扩散范式融合为 block-wise hybrid——全局使用自回归保持因果结构和 KV caching，局部使用扩散实现并行解码。关键发现：(a) AR 模型的训练效率远高于纯扩散模型；(b) 轻量级范式转换（lightweight paradigm conversion）即可将训练好的 AR 模型转换为 block-wise diffusion 模型；(c) 30B MoE 模型在科学推理任务上超越纯 AR 对应模型。这直接证明了范式融合不仅在理论上有吸引力，在实践中也能产生优于单一范式的结果。

2. **GarmentGPT (ICLR 2026)**：本身就是范式融合的实例——将 VQ-VAE（表示学习范式）与 VLM 自回归生成结合。RVQ-VAE 在表示层面解决了连续坐标回归的困难，VLM 在生成层面提供了 compositional reasoning 能力。Panel Accuracy 95.62% 的结果表明这种融合是有效的。

3. **GarmageNet (SIGGRAPH Asia 2025)**：将图像化表征（VAE latent encoding）与扩散生成结合，并通过 GarmageJigsaw 模块引入了专门的缝合关系推理。这种"生成+后处理"的架构实质上是范式融合——用扩散处理几何生成，用判别式模型处理拓扑推理。

### 版型生成的范式融合可能性

| 融合方案 | 组合 | 各范式负责的子任务 | 理论优势 | 现有验证 |
|---------|------|-----------------|---------|---------|
| **程序合成 + 扩散细化** | 程序合成 → 扩散模型 | 程序生成粗糙参数化版型 → 扩散模型细化几何细节 | 结构正确性 + 几何精度 | 无直接验证 |
| **VQ-VAE + AR 生成** | RVQ-VAE tokenization → VLM AR | VQ-VAE 压缩表示空间 → VLM 进行 compositional reasoning | 高效表示 + 语言推理 | GarmentGPT 已验证 |
| **Garmage + 程序合成** | GarmageNet 生成 → 程序逆向提取 | 扩散生成 3D 几何 → 程序化参数提取实现可编辑性 | 几何保真 + 参数化可编辑 | 无直接验证 |
| **Block-wise AR-Diffusion** | SDAR 架构 | 全局 AR 保持版型结构顺序 → 局部扩散并行生成 panel 细节 | 结构连贯 + 并行效率 | SDAR 在 NLP 已验证，版型领域未验证 |
| **两阶段：拓扑 + 几何** | 分类器/图生成 → 任意几何生成器 | 先预测拓扑结构 → 再在固定拓扑下生成几何 | 问题分解降低难度 | 无直接验证，但设计思路清晰 |

### 缺失的理论基础

1. **最优分解问题**：版型生成任务应如何分解为子任务？哪些子任务由哪种范式处理最优？目前没有理论框架来回答这个问题，所有融合方案都是启发式的。

2. **信息传递接口**：不同范式之间如何传递中间表示？程序合成输出的 GarmentCode 程序与扩散模型的 latent space 之间缺乏自然的转换接口。GarmentGPT 的 RVQ-VAE 提供了一种可能的桥梁（连续 → 离散 → 连续的转换路径），但尚未被系统性地利用。

3. **端到端训练 vs 级联训练**：SDAR 证明了通过轻量级适应将 AR 模型转换为 hybrid 模型的可行性。但对于版型生成这种结构化输出任务，端到端训练融合模型与级联使用多个模型相比，哪种策略更优，缺乏比较研究。

4. **约束传播的范式依赖性**：不同范式对版型约束（边长匹配、stitch 一致性、可缝制性）的保证机制根本不同——程序合成通过程序逻辑硬编码约束，扩散模型通过训练数据隐式学习约束，AR 模型通过 sequential dependency 传递约束。融合后的系统如何协调这些不同的约束保证机制，是一个开放的研究问题。

### 证据来源

1. SDAR (Cheng et al., 2025)：30B 参数的 AR-Diffusion hybrid，open-sourced models 1.7B-30B
2. GarmentGPT (Weng et al., ICLR 2026)：RVQ-VAE + VLM 融合，Panel Acc 95.62%
3. GarmageNet (SIGGRAPH Asia 2025)：VAE + DiT + GarmageJigsaw 的多模块融合
4. GarmentDiffusion (IJCAI 2025)：纯扩散范式，提供了单范式的 performance baseline

### 影响程度评估

**高（High）** — 范式融合是版型生成领域从"各自为战的范式探索"走向"系统性的最优方案设计"的关键转折点。SDAR 在 NLP 领域的成功和 GarmentGPT 在版型领域的初步验证表明，融合方案确实可以超越单一范式。然而，当前的融合尝试都是特定的、启发式的，缺乏指导未来融合设计的理论框架。这是一个真正开放的研究问题，具有高学术价值和实践意义。

---

## 附录：原 Gap 12（编辑一致性）处理说明

v0 的 Gap 12（编辑一致性——局部修改导致全局破坏）与方向二 Gap 8 存在高度重叠（相同的问题定义、相同的解决方案空间）。接受 Rebuttal 建议，该内容已合并至方向二的 Gap 8 中统一讨论。此处不再重复。

关键要点保留：
- ChatGarment 论文 §Conclusion 明确承认局部编辑导致全局变化的问题
- 参数化 CAD 约束传播是 30 年历史的成熟技术（SolidWorks, Fusion 360），可直接应用于版型编辑的约束一致性保证
- 各范式都有部分但不完美的编辑一致性机制，需要的是增强而非从零建立
- Autodesk Research (2025) "Aligning Constraint Generation with Design Intent in Parametric CAD" 提供了将设计意图与约束传播对齐的最新研究方向
