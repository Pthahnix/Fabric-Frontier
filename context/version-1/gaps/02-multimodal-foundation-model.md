<!-- markdownlint-disable -->
# 方向二：多模态基础模型 for 版型生成（v1）

## v0→v1 变更说明

| 变更类型 | 内容 |
|---------|------|
| **Gap 5 降级** | 从"极高（Critical）"降至"高（High）"。GarmageSet（14,801件专业设计师版型）的发布直接打破了"全部合成数据"的论断；DressWild的VLM引导数据增强和NGL-Prompter的training-free范式进一步缓解了对合成数据的依赖。同时采纳rebuttal观点：sim-to-real gap在机器人等领域已有成熟迁移方案，版型的2D几何迁移难度低于3D物理仿真迁移 |
| **Gap 6 细化** | 维持"高（High）"但修正分析。NGL-Prompter已实现multi-layer outfit恢复（逐层独立但VLM可推理遮挡部分），但确认层间物理交互仍完全缺失。采纳rebuttal"顺序生成+约束"路径的合理性，但指出工业多层数据匮乏仍是核心瓶颈 |
| **Gap 7 升级** | 从"中高（Medium-High）"升至"高（High）"。采纳rebuttal核心论点：QuantiPhy/DeepPHY证据表明VLM定量推理存在系统性缺陷，问题不仅是"语义歧义"而是"定量几何推理能力不足"。NGL-Prompter的NGL中间语言实证验证了结构化中间表示的有效性，但同时暴露了VLM在细粒度属性上的精度瓶颈 |
| **Gap 8 降级** | 从"高（High）"降至"中（Medium）"。采纳rebuttal核心论点：参数化CAD约束传播是30年成熟技术；GarmentCode/ChatGarment的参数化框架天然支持约束传播；CAD-Assistant的tool augmentation方案已在CAD领域验证。问题本质是工程集成而非开放研究 |
| **新增 Gap 9** | 新增"版型表示范式的根本分歧"。GarmageNet的Garmage表示（geometry image + point-wise sewing）与GarmentCode的参数化DSL表示代表了两条截然不同的技术路线，二者在表达力、可编辑性、数据依赖上各有本质优劣，形成方向二内部的根本性技术选择问题 |
| **新增 Gap 10** | 新增"面料物理属性估计的缺失"。Image2Garment (2026)揭示了一个被忽视的关键gap：现有方法只生成几何版型，完全忽略面料物理参数（弯曲刚度、拉伸模量、密度等），导致生成结果无法进行物理真实的仿真 |
| **证据更新** | 全面纳入2025-2026新文献：GarmageNet/GarmageSet (SIGGRAPH Asia 2025)、NGL-Prompter (2026)、DressWild (2026)、Image2Garment (2026)、GarmentDiffusion (IJCAI 2025)、GenPattern (2025)、SewingLDM (ICCV 2025) |

---

## 概述

随着大规模多模态基础模型（如GPT-5、Qwen3-VL、LLaVA）的快速发展，将这些模型应用于版型生成已成为当前最活跃的研究方向。其核心理念是：利用基础模型强大的视觉理解和语言理解能力，直接从图像或文本描述中推断版型结构。2025-2026年间涌现了大量代表性工作：AIpparel（多模态tokenizer + GPT式架构）、ChatGarment（微调VLM生成DSL参数）、DressWild（从野外图像恢复版型）、GarmageNet（geometry image统一表示 + diffusion生成）、NGL-Prompter（training-free VLM prompting）、GarmentDiffusion（多模态diffusion transformer）等。

然而，这一方向在取得显著进展的同时，也暴露了更深层次的问题：版型表示范式的根本分歧（参数化DSL vs 几何图像）、VLM定量推理的系统性缺陷、面料物理属性的完全缺失、以及真实工业数据的持续匮乏。

---

## Gap 5: 数据集偏差——合成 vs 真实（v1修订）

### 问题描述

v0版本声称"当前所有大规模数据集都是程序化合成的，没有任何包含真实版型"。这一论断在v1中需要**重大修正**：GarmageSet的发布（Style3D, SIGGRAPH Asia 2025）提供了14,801件由**专业版型师设计的工业级版型**，其数据来源于真实生产工作流而非程序化生成。这是领域内首个达到工业品质的大规模版型数据集，直接打破了v0的核心论断。

然而，数据偏差问题并未完全消解——它的性质发生了转变：从"纯合成 vs 无真实"变为"数据分布覆盖度不足 + 多源数据融合困难"。

### 现有方法的局限

**数据集格局的重大变化（2025-2026更新）**：

| 数据集 | 规模 | 输入模态 | 来源 | 真实版型 | 新增信息 |
|--------|------|----------|------|----------|----------|
| DressCode (SIGGRAPH 2024) | 20.3K | 文本 | GarmentCode程序化生成 | 否 | — |
| SewFactory (2024) | 19.1K | 图像 | 程序化生成 | 否 | — |
| GarmentCodeData (GCD) | 130K | 图像 | GarmentCode程序化生成 | 否 | — |
| GCD-MM / AIpparel | 120K | 文本+图像+编辑 | GarmentCode程序化生成 | 否 | — |
| **GarmageSet (2025)** | **14,801** | **文本+草图+点云+版型** | **专业版型师设计** | **是** | 来自Style3D真实生产工作流 |
| DressWild数据集 (2026) | 20K+ | 图像（VLM增强多视角多姿态） | GarmentCode+VLM增强 | 否 | VLM引导数据增强 |

**关键观察（v1修订）**：

1. **GarmageSet打破了"全合成"局面**：GarmageSet的14,801件版型源自专业版型师在Style3D工业软件中的设计，包含多边缘环面片（multi-edge-loop panels）、复杂轮廓（如螺旋形）、多对多的部分边缘缝合映射——这些特征在GarmentCode程序化数据中完全不存在。GarmageNet论文明确指出GarmentCodeData"substantially simplifies realities of production data"。
2. **但GarmageSet也有其局限**：(a) 14.8K的规模相对130K的GCD仍然较小；(b) 所有版型均着装于单一标准体型的A-pose mannequin（亚洲S码84cm胸围），缺乏体型多样性；(c) 论文自承"lacking body shape and pose variance"为主要局限。
3. **合成数据偏差的根源被进一步澄清**：NGL-Prompter论文明确指出GarmentCode的随机参数采样"underconstrained"——随机参数组合常生成不现实的服装（如不对称上衣的左右部分不构成合理设计），导致训练数据缺乏真实世界服装参数之间的相关性（如T恤通常同时具有圆领和短袖）。这意味着sim-to-real gap的核心不仅是"合成vs真实"，更是**采样策略导致的分布不匹配**。
4. **VLM引导的数据增强提供了新路径**：DressWild利用VLM（Nanobanana Pro）从标准T-pose图像合成多视角、多姿态变体，将输入多样性扩展到10种视角 × 21种姿态 × 21种手势的组合空间。这种方法不改变底层版型数据的分布，但显著增强了模型对姿态/视角变化的鲁棒性。

**采纳rebuttal的观点**：

- **Sim-to-real迁移在其他领域已有成功案例**：robotics领域的Domain Randomization (Tobin et al., 2017)、Isaac Sim等成功实现了3D物理仿真到真实世界的策略迁移。2D版型图纸的sim-to-real gap在几何维度上确实低于3D物理世界的gap。新的实证数据支持这一观点：sim-to-real cloth manipulation benchmark (Blanco-Mulero et al., 2023) 系统量化了布料仿真器（MuJoCo, Bullet, Flex, SOFA）与真实世界之间的gap，为缩小差距提供了量化基准。
- **真实数据源确实存在但未被系统收集**：FreeSewing.org、Etsy/Patreon的DXF版型、服装院校教材——问题是数据工程而非数据稀缺。GarmageSet的出现证明了工业合作路径的可行性。
- **问题根源可能在DSL表达力**：如rebuttal所述，GarmentCode的DSL表达力限制了合成数据的多样性。GarmageNet通过完全绕过GarmentCode、直接使用工业版型，从另一个角度验证了这一判断。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| GarmageNet (SIGGRAPH Asia 2025) | GarmageSet: 14,801件专业设计版型，明确指出GarmentCodeData"substantially simplifies realities of production data" |
| NGL-Prompter (2026) | GarmentCode随机采样"underconstrained"，缺乏真实服装参数相关性 |
| DressWild (2026) | VLM引导数据增强（10视角×21姿态×21手势） |
| Benchmarking Sim-to-Real Gap in Cloth (2023) | 系统量化布料仿真器与真实世界的gap |
| GarmentLab (2024) | 提出sim-to-real算法和真实世界benchmark |
| Image2Garment (2026) | FTAG数据集从在线服装目录收集面料属性标注，证明真实标注数据可规模化获取 |

### 影响程度评估

**高（High）**（v0为"极高Critical"，v1降级）。GarmageSet的发布将该gap从"完全无真实数据"降级为"真实数据规模和多样性仍需扩展"。但以下问题仍然严峻：

1. **跨数据集泛化**：在GarmentCodeData上训练的模型（如ChatGarment、AIpparel）是否能泛化到GarmageSet的复杂版型？反之亦然？当前无任何工作系统评估跨数据集性能。
2. **体型多样性缺失**：GarmageSet仅使用单一标准体型，GarmentCodeData虽支持SMPL参数化体型但仅限合成。真实世界的个体体型差异对版型的影响未被任何数据集捕获。
3. **评估指标仍在合成分布内**：当前主要benchmark（Dress4D、CloSE）的ground truth来自3D扫描而非工业版型，无法评估生成版型的工业可用性。

**潜在解决路径**（v1更新）：
1. **GarmageSet路径的扩展**：与更多版型工作室合作，将Style3D模式推广到更大规模、更多体型
2. **混合训练策略**：在大规模合成数据上预训练 + 在小规模真实数据上微调（类似Image2Garment的两阶段策略）
3. **VLM引导的数据增强**：利用VLM生成多样化的条件输入（多视角、多姿态），在不改变版型分布的前提下提升输入鲁棒性
4. **众包数据工程**：系统收集FreeSewing、Etsy等平台的开源版型数据，建立版型领域的"ImageNet"

---

## Gap 6: 多层服装的联合生成（v1修订）

### 问题描述

真实穿着场景中的多层服装联合生成仍然是一个重要挑战，但v1需要修正v0的分析：NGL-Prompter (2026)已经实现了multi-layer outfit的版型恢复，尽管其方式是逐层独立处理。核心gap已从"完全无法处理多层"转变为"缺乏层间物理交互建模"。

### 现有方法的局限

**2025-2026进展更新**：

- **NGL-Prompter (2026)**：实现了multi-layer outfit恢复。其方法是：先由VLM识别所有可见服装层及其从内到外的排序，然后逐层独立处理。由于VLM具备推理部分遮挡服装的能力，该方法可以从单张图像中恢复被外层服装遮挡的内层服装。论文宣称这是"the first training-free approach...capable of handling both single-layer and multi-layer garments"。
- **DressWild (2026)**：论文Application部分展示了"Multi-layer Garment Generation"能力，但实际处理方式仍是逐件分别预测。
- **GarmageNet (2025)**：自承"Multi-layer designs and small-panel handling"为主要局限之一。GarmageSet虽然包含工业级版型，但其数据中的多层构造信息有限。
- **Dress-1-to-3 (2025)**：使用CIPC（Codimensional Incremental Potential Contact）处理多层布料碰撞检测，证明物理仿真领域已有现成的层间碰撞解决方案。

**仍然缺失的关键能力**：

1. **层间松量自适应**：外套的版型需要根据内层服装的体积自动调整松量（ease）。当前所有方法都忽略这一点——NGL-Prompter逐层独立推理意味着外套的生成不知道内层衬衫的版型细节。
2. **层间物理交互建模**：面料间摩擦、重力导致的层间压迫、穿着顺序对最终形态的影响——这些在工业版型设计中需要考虑的因素完全未被建模。
3. **多层训练数据**：工业技术包（tech pack）包含所有层的完整版型，但几乎不以数字化格式公开。GarmageSet虽为工业级，但论文未提及多层版型的系统性收录。

**采纳rebuttal的观点**：

- **顺序生成+约束比联合生成更实际**：rebuttal提出的"先生成外壳 → 推导里布（外壳-缝份-转折余量）→ 推导衬布"的工程路径在实践中合理。层间关系在工艺上确实有明确规则，不一定需要从数据中隐式学习。
- **瓶颈是训练数据而非架构**：确认这一判断。NGL-Prompter用training-free方式绕过了训练数据问题，但代价是无法建模精细的层间关系。
- **参数化方法天然适合多层**：GarmentCode理论上可以通过程序逻辑添加多层支持，但实际上没有任何工作实现这一扩展。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| NGL-Prompter (2026) | 首个支持multi-layer outfit恢复的方法（逐层独立+VLM推理遮挡） |
| DressWild (2026) | Application展示multi-layer但逐件独立处理 |
| GarmageNet (2025) | 自承multi-layer designs为主要局限 |
| Dress-1-to-3 (2025) | CIPC碰撞检测验证层间物理仿真可行性 |
| ChatGarment (CVPR 2025) | 所有结果为单件服装 |

### 影响程度评估

**高（High）**（维持v0评级，但修正分析方向）。NGL-Prompter的多层恢复能力标志着该领域的重要进步，但"逐层独立处理"与"层间物理感知"之间的gap仍然显著。影响集中在：

1. **外套/套装设计场景**：逐层独立生成可能导致松量不匹配（外套太紧或太松）
2. **虚拟试穿的完整穿搭需求**：需要多件服装在同一人体上的协调展示
3. **工业版型设计的里布/衬布推导**：这是日常工业需求，当前完全无自动化支持

**潜在方向**（v1更新）：
1. **NGL-Prompter式分层推理 + 约束后处理**：VLM逐层恢复 → 规则化约束引擎调整层间松量
2. **扩展GarmageSet收录多层数据**：与Style3D合作，系统性收录外壳+里布+衬布的完整技术包
3. **Dress-1-to-3的CIPC作为后处理模块**：生成后通过物理仿真检测和修正层间穿透

---

## Gap 7: VLM定量几何推理的系统性缺陷（v1重构）

### 问题描述

v0将此gap定义为"文本描述的语义歧义"。v1采纳rebuttal的核心论点，**将问题重新定义为更深层次的VLM定量推理能力缺陷**。语义歧义是问题的表层表现，而QuantiPhy (2025)、DeepPHY (2025)的基准测试揭示了VLM在定量物理推理上的系统性能力不足——这比单纯的"歧义消解"更加根本和棘手。

同时，NGL-Prompter (2026)的实证结果提供了**支持和限制这一gap**的双重证据：一方面验证了结构化中间表示（NGL）能显著提升VLM的服装理解能力；另一方面也暴露了VLM在细粒度属性预测上的精度天花板。

### 现有方法的局限

**VLM定量推理的系统性证据**：

1. **QuantiPhy (2025)**：评估了最先进VLM的定量物理推理能力，最佳模型仅达53.1% MRA，低于人类的55.6%。更关键的是：在counterfactual设置下MRA暴跌80%，论文结论为"VLMs do not reason but memorize"。Chain-of-Thought (CoT) prompting在21个模型中的19个上**反而降低了性能**。
2. **DeepPHY (2025)**：更直接的结论——"even state-of-the-art VLMs struggle to translate descriptive physical knowledge into precise, predictive control"，且"most open-source models cannot surpass MOCK (random) results"。
3. **NGL-Prompter的实证验证**：
   - **VLM擅长识别常见的服装描述属性**（如领口类型、袖子有无、前开襟），在NGL-0层级（27个基础参数）上大模型表现良好
   - **VLM在细粒度细节上表现显著下降**：NGL-1层级（46个参数，含更精细的款式细节）的准确率明显降低。论文指出"current VLMs still require additional cues to reliably capture finer-grained details at higher LODs"
   - **VLM无法直接回归GarmentCode的数值参数**：这是NGL设计的核心动机——"VLMs are effective at describing garments in natural language, yet perform poorly when asked to directly regress GarmentCode parameters from images"
   - **连续数值被离散化**：NGL将所有连续参数转化为离散的自然语言描述（如length = mini | midi | maxi），本质上是**回避而非解决**了定量精度问题

**语义歧义的持续存在**（v0关注点仍然有效）：

- **DressCode / SewingGPT**的GPT-4V生成文本标注使用非量化描述性语言
- **GarmentDiffusion (IJCAI 2025)**重新设计标注流程使用Llama-3.1-8B生成brief + detailed双层描述，但仍缺乏数值锚定
- **ChatGarment**的语义解析完全依赖LLM，无跨文化适配机制
- **GenPattern (2025)**提出dual-graph增强的版型生成，将结构化图建模引入MLLM管线，但graph representation能否真正解决歧义尚需验证

**领域微调的缓解但非根治**：

rebuttal引用SVGen (2025)论证领域微调可以显著提升VLM在特定任务上的精度。这一观点有道理，但需要注意关键区别：SVGen处理的是**结构化矢量图形**（有明确的几何规则），而服装版型涉及的是**设计意图到几何参数的模糊映射**（"宽松"到底是多少cm取决于品牌、文化、个人偏好）。领域微调可以学习统计相关性，但无法消除本质上的多义性。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| QuantiPhy (2025) | 最佳VLM定量物理推理53.1% MRA, counterfactual暴跌80%, CoT在19/21模型上有害 |
| DeepPHY (2025) | "most open-source models cannot surpass MOCK results"在物理参数预测上 |
| NGL-Prompter (2026) | VLM无法直接回归GarmentCode数值参数；NGL-1细粒度准确率显著下降 |
| GarmentDiffusion (IJCAI 2025) | 指出DressCode的"coarse sewing pattern descriptions"，重设计标注但仍无数值锚定 |
| N3D-VLM (2025) | 结构化3D grounding将空间推理准确率从36.3%提升到92.1% |
| Image2Garment (2026) | 通过语义中间表示（面料属性→物理参数）成功规避了VLM直接预测数值的弱点 |

### 影响程度评估

**高（High）**（v0为"中高Medium-High"，v1升级）。采纳rebuttal的论证，VLM的定量推理缺陷比v0的"语义歧义"判断更加严重。NGL-Prompter虽然通过离散化中间语言绕过了直接数值回归，但代价是精度上限——离散化的"midi length"无法表达"裙长恰好到膝盖下方5cm"这样的精确需求。

**核心矛盾**：工业版型需要毫米级精度，而VLM的自然语言理解天然是模糊的、定性的。这不是通过更好的prompt engineering或更大的模型就能消解的矛盾。

**潜在解决路径**（v1更新，综合rebuttal和新证据）：
1. **N3D-VLM式结构化中间步骤**：VLM先映射到结构化中间表示 → 参数查询表 → constraint solver精确化。NGL-Prompter的NGL已经实现了这一架构的前半段
2. **Image2Garment式语义分解**：将问题分解为"图像→语义属性"（VLM擅长）+"语义属性→数值参数"（小型专用模型+查找表），规避VLM直接预测数值的弱点
3. **交互式消歧**（Chat2Layout验证的路径）：不追求一次性完美理解，通过多轮交互逐步精化。NASA-TLX评估显示用户体验可接受
4. **领域微调 + GRPO强化学习**：SVGen展示的路径——在精确"描述↔参数"配对数据上微调小型模型，可能超越通用大模型

---

## Gap 8: 版型编辑的语义一致性（v1修订）

### 问题描述

v0将版型编辑的全局一致性定义为一个"高"级别的研究问题。v1采纳rebuttal的核心论点：**对于基于GarmentCode等参数化框架的方法，编辑一致性本质上是一个工程集成问题，而非开放性研究问题**。参数化CAD系统中的约束传播技术有30年以上的工程历史，SolidWorks、CATIA等系统每天处理数百万次参数化编辑。

然而，对于非参数化方法（如GarmageNet的geometry image表示、基于vector quantization的方法），编辑一致性仍然是一个有意义的研究挑战。

### 现有方法的局限

**参数化方法的编辑一致性（工程问题，已有成熟方案）**：

- **ChatGarment (CVPR 2025)**：虽然论文承认"altering the skirt length might unintentionally affect other parts"，但这是当前实现的局限而非范式的局限。GarmentCode作为参数化程序，参数之间的依赖关系可以通过程序逻辑显式编码。问题在于ChatGarment的VLM直接修改JSON参数而未调用约束传播引擎。
- **CAD-Assistant (2025)**：展示了tool augmentation + iterative state inspection的方案——每次修改后通过tool call检查全局状态并修正。这一架构可以直接迁移到版型编辑场景。
- **GarmentCode/GarmentCodeRC本身**：参数化程序天然支持依赖关系——修改一个参数时，程序逻辑自动计算所有受影响的输出。问题是当前AI方法（ChatGarment、Design2GarmentCode）绕过了程序的约束检查，直接输出参数值。

**非参数化方法的编辑一致性（仍有研究挑战）**：

- **AIpparel (2025)**：token级别的面片编辑，修改一个面片的token不自动触发关联面片调整。由于其表示不是参数化的（而是序列化的token），无法直接应用CAD约束传播。
- **GarmageNet (2025)**：基于geometry image的表示使版型编辑需要在图像空间进行。progressive editing在语义层面可行（通过文本指令编辑），但面片间的几何约束（缝合边匹配）需要额外的GarmageJigsaw模块重新推断。
- **GarmentDiffusion (IJCAI 2025)**：支持从incomplete sewing pattern补全，但未展示迭代编辑能力。

**版本管理的持续缺失**：
所有AI方法仍然没有undo/redo或版本管理概念。这在交互式设计场景中是一个实际的用户体验问题，但本质上是软件工程问题而非AI研究问题。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| ChatGarment (CVPR 2025) | §Limitations: "altering the skirt length might unintentionally affect other parts" |
| CAD-Assistant (2025) | Tool augmentation + iterative state inspection在CAD领域验证有效 |
| GarmageNet (2025) | Progressive editing通过文本指令实现，但需GarmageJigsaw重建缝合关系 |
| 工业CAD系统 | D-Cubed DCM等几何约束求解器有30年工程历史 |
| Chat2Layout (2024) | Visual self-reflection机制将OOB错误从24.8%降至21.0% |

### 影响程度评估

**中（Medium）**（v0为"高High"，v1降级）。对于占据当前主流的参数化方法（ChatGarment、Design2GarmentCode），编辑一致性通过集成CAD约束传播技术即可解决，是一个工程任务而非研究瓶颈。对于非参数化方法（AIpparel、GarmageNet），编辑一致性仍有研究价值，但这些方法正在向版型编辑以外的优势方向发展（如几何精度、复杂形态生成）。

**潜在解决路径**（v1更新）：
1. **参数化方法**：集成CAD约束传播引擎（已有成熟商业方案），在VLM输出参数后增加约束检查和自动修正步骤
2. **非参数化方法**：CAD-Assistant式的"编辑→全局检查→修复"循环；或Chat2Layout式的visual self-reflection
3. **混合方案**：使用参数化框架确保结构一致性，使用非参数化方法处理自由形变编辑

---

## Gap 9: 版型表示范式的根本分歧（v1新增）

### 问题描述

2025-2026年的研究揭示了方向二内部一个未被充分讨论的根本性分歧：**版型应该用什么表示？** 当前存在三条截然不同的表示范式，各有本质优劣，选择哪条路线将深刻影响整个系统的能力边界。

### 三大表示范式的对比

| 维度 | 参数化DSL（GarmentCode系列） | 序列化Token（AIpparel/DressCode系列） | 几何图像（GarmageNet） |
|------|----------------------------|--------------------------------------|----------------------|
| **代表方法** | ChatGarment, Design2GarmentCode, NGL-Prompter | AIpparel, DressCode, SewFormer, GarmentDiffusion | GarmageNet |
| **表达力** | 受限于DSL定义的设计空间 | 理论上无限，但受token化精度限制 | 理论上无限，像素级精度 |
| **结构正确性保证** | **程序逻辑保证**（缝合、对称、比例） | **无保证**（可能生成不可缝制的版型） | 需要GarmageJigsaw后处理推断缝合关系 |
| **可编辑性** | **极强**（修改参数即可） | 弱（需要重新生成） | 中等（可通过条件编辑） |
| **数据依赖** | 低（参数空间可程序化采样） | 高（需要大量token化版型数据） | 高（需要工业级版型数据） |
| **工业兼容性** | 中（输出参数化程序，需要DSL解析器） | 低（输出token序列，需要后处理） | **高**（输出三角网格+缝合关系，直接兼容仿真器） |
| **复杂形态** | **差**（超出DSL范围的设计无法表达） | 中等 | **强**（vertex-level precision, 螺旋/褶皱等） |

**关键分歧的深层含义**：

1. **GarmageNet vs GarmentCode的核心矛盾**：GarmageNet论文明确指出GarmentCodeData在三个方面"substantially simplifies realities of production data"——(a) 工业版型包含multi-edge-loop panels和复杂轮廓，GarmentCode只支持single-edge-loop和四种边类型；(b) 工业缝合需要many-to-many、partial-edge映射，GarmentCode只编码one-to-one full-edge对应；(c) GarmentCode提供per-panel rigid placement用于直接着装，而工业DXF版型通常不包含缝合图或rigid placement。这意味着**GarmentCode系列方法在表达力上存在硬性天花板**。

2. **但参数化方法的结构保证无可替代**：GarmageNet需要GarmageJigsaw模块来后处理推断缝合关系（准确率虽高但非100%），而GarmentCode的程序逻辑**天然保证**结构正确性。在工业应用中，一个缝合关系推断错误就意味着服装无法制作。

3. **NGL-Prompter揭示了一个有趣的中间地带**：NGL将GarmentCode参数重新组织为VLM友好的自然语言描述，然后通过确定性解析器映射回GarmentCode。这说明参数化DSL可以通过中间层适配来缓解其VLM-unfriendly的问题，而不必放弃结构保证。

4. **GenPattern (2025)的dual-graph方法**：将版型表示为图结构（面片→节点，缝合→边），用MLLM + graph neural network联合处理。这是参数化DSL和自由几何之间的第三条路线——保留结构关系的同时不受DSL表达力限制。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| GarmageNet (SIGGRAPH Asia 2025) | Garmage表示: geometry image + point-wise sewing，明确指出GarmentCode的三大局限 |
| NGL-Prompter (2026) | NGL中间语言连接自然语言与GarmentCode，验证中间表示层的价值 |
| GarmentDiffusion (IJCAI 2025) | Diffusion transformer + 多模态输入生成向量化版型，序列化方法的新高度 |
| GenPattern (2025) | Dual-graph + MLLM方法，图结构作为版型的第四种表示范式 |
| SewingLDM (ICCV 2025) | 多模态latent diffusion生成版型，进一步扩展序列化方法的能力 |

### 影响程度评估

**高（High）**。这不是一个可以通过增量改进解决的gap——它是方向二内部的**架构选择问题**，决定了整个系统的能力上限和工业适用性。选择参数化DSL意味着牺牲表达力换取结构保证；选择几何图像意味着获得最大灵活性但需要额外机制保证可制造性；选择序列化token则在两者之间取得一个不确定的平衡。

**对FABRIC-FRONTIER的特殊影响**：该项目需要在早期做出表示范式的战略选择，这一选择将贯穿整个系统架构。建议：

1. **短期（原型阶段）**：基于GarmentCode参数化方法，利用NGL-Prompter式中间层适配VLM，最大化结构保证
2. **中期（产品化阶段）**：引入GarmageNet式几何表示处理超出DSL范围的复杂设计，用GarmageJigsaw + 约束检查器保证可制造性
3. **长期**：探索GenPattern式图结构表示，在结构保证和表达灵活性之间取得最优平衡

---

## Gap 10: 面料物理属性估计的缺失（v1新增）

### 问题描述

当前所有版型生成方法——无论采用何种表示范式——都只生成**几何版型**（面片形状、缝合关系、3D placement），完全忽略了面料的**物理属性**（弯曲刚度bending stiffness、拉伸模量stretch modulus、密度density、阻尼damping等）。然而，同一套版型在不同面料上的仿真结果截然不同（Image2Garment论文Figure 2直观展示了wool、cotton-cork、polyester三种面料在相同动作序列下的巨大差异）。这意味着当前方法生成的"simulation-ready"版型实际上缺少一半的关键信息。

### 现有方法的局限

- **ChatGarment (CVPR 2025)**：生成GarmentCode参数 → 版型几何。面料属性使用仿真器的默认值，与用户输入图像中的实际面料无关。
- **AIpparel (2025)**：token化版型的几何信息。无面料属性预测。
- **GarmageNet (2025)**：Garmage编码几何但不编码物理属性。论文未讨论面料参数。
- **DressWild (2026)**：预测版型参数用于仿真，但面料属性固定。
- **Image2Garment (2026)**：**唯一系统性处理面料属性的工作**。提出两阶段方法：(1) VLM从图像预测面料属性（材质组成、织物类型、结构类型、厚度、重量）；(2) 轻量级映射器将属性转化为仿真器物理参数。关键数据集：FTAG（图像→面料属性标注，来自在线服装目录）和T2P（面料属性→物理参数，来自工业仿真器测量）。

**根本挑战**：

1. **图像→物理参数的ill-posedness**：从单张图像直接预测面料物理参数是高度病态的（drape外观对多个物理参数有相似的响应）
2. **面料属性数据的稀缺**：将面料描述（如"100% cotton, plain weave"）映射到精确物理参数（弯曲刚度、剪切模量等）需要系统性的材料测试数据，目前仅Image2Garment的T2P数据集开始填补这一空白
3. **面料属性与版型设计的耦合**：真实版型师会根据面料特性调整版型参数（如薄透面料需要更多缝份、厚面料需要减少层叠），当前没有任何方法建模这种耦合关系

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| Image2Garment (2026) | 首个系统性处理面料物理属性的工作；FTAG和T2P数据集；展示面料参数对仿真结果的决定性影响 |
| ChatGarment (CVPR 2025) | 仿真使用默认面料参数，与实际面料无关 |
| GarmageNet (2025) | Garmage不编码物理属性 |
| Textile IR (2026) | 提出physics-aware fashion CAD的中间表示，讨论面料属性在版型设计中的角色 |

### 影响程度评估

**中高（Medium-High）**。对于"生成视觉上合理的3D服装"这一目标，面料属性的缺失影响有限（使用合理的默认值即可获得视觉上可接受的结果）。但对于"生成物理真实的、可制造的版型"这一更高目标，面料属性是不可或缺的——不同面料需要不同的缝份、不同的省道深度、不同的松量设计。

**潜在解决路径**：
1. **Image2Garment路径的推广**：将其两阶段方法（VLM→面料属性→物理参数）集成到其他版型生成框架中
2. **面料属性作为条件输入**：在版型生成时将面料属性作为额外的条件信号，训练模型学习面料-版型的耦合关系
3. **扩展训练数据**：在GarmageSet等数据集中增加面料物理属性标注

---

## 总结：方向二研究差距全景（v1）

| Gap | 核心问题 | v0评级 | v1评级 | 变化原因 | 短期可解性 |
|-----|----------|--------|--------|----------|------------|
| Gap 5: 数据集偏差 | 合成数据主导，真实数据规模/多样性不足 | 极高 | **高** | GarmageSet提供14.8K工业版型；sim-to-real有成熟迁移方案 | 中（需扩展GarmageSet规模） |
| Gap 6: 多层联合生成 | 逐层独立处理，缺乏层间物理交互 | 高 | **高** | NGL-Prompter实现逐层恢复但无层间交互；瓶颈是数据而非架构 | 中（顺序生成+约束路径可行） |
| Gap 7: VLM定量推理缺陷 | VLM在定量几何推理上系统性不足 | 中高 | **高** | QuantiPhy/DeepPHY证据；NGL-Prompter验证并暴露了VLM精度天花板 | 低（需要架构创新而非增量改进） |
| Gap 8: 编辑一致性 | 参数化方法可解，非参数化方法仍有挑战 | 高 | **中** | 参数化CAD约束传播是30年成熟技术；本质是工程集成问题 | 高（集成CAD约束引擎即可） |
| **Gap 9: 表示范式分歧** | 参数化DSL vs token序列 vs 几何图像 | — | **高** | GarmageNet vs GarmentCode揭示根本性表达力-结构保证trade-off | 低（需要战略选择而非技术突破） |
| **Gap 10: 面料物理属性** | 只生成几何版型，忽略面料物理参数 | — | **中高** | Image2Garment首次系统处理；对物理真实仿真不可或缺 | 中（Image2Garment路径可推广） |

### 核心洞察（v1更新）

v0的核心洞察——"多模态基础模型在理解服装和生成版型之间缺少关键的中间层：服装设计知识的结构化表示"——在v1中得到了**实证验证和深化**：

1. **NGL-Prompter验证了结构化中间表示的价值**：NGL作为VLM与GarmentCode之间的桥梁，将离散化的自然语言属性映射为参数化版型，在training-free设置下达到SOTA性能。这证明了"显式编码服装设计领域知识"的路线正确性。

2. **但问题比预想的更深**：VLM的定量推理缺陷不仅是"缺少中间层"的问题，而是模型架构层面的系统性弱点。即使有了NGL这样的中间表示，VLM仍然无法可靠地推断细粒度的几何参数。这意味着需要**混合架构**——VLM负责高层语义理解，专用模块负责精确的几何推理和约束满足。

3. **表示范式的选择比任何单个技术问题都更根本**：GarmageNet（工业级几何精度但需后处理保证结构性）vs GarmentCode（结构保证但表达力受限）的分歧，本质上反映了服装版型的固有二元性——它既是参数化的设计意图，又是自由形态的几何对象。未来的突破性方法可能需要统一这两个视角。

4. **面料物理属性是被集体忽视的关键维度**：当前所有方法都在追求更好的几何精度，但忽略了面料物理参数对最终仿真结果和实际制造的决定性影响。Image2Garment的语义分解策略（图像→面料属性→物理参数）提供了一条务实的解决路径。
