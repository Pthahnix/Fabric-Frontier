<!-- markdownlint-disable -->
# 方向一：文本/图像 → 参数化版型程序合成（v1）

## v0→v1 变更说明

本文档基于 v0 Gap 分析及其 Rebuttal 进行了系统性修订。主要变更如下：

### 保留与强化的结论
- **Gap 1（DSL 表达力瓶颈）**：维持"极高"评级。Rebuttal 提出的 LILO library learning 方案经验证存在根本性局限——两篇独立研究（Berlot-Attwell et al., 2024; 2025）证明 LLM-based library learning 的实际复用率极低，性能提升主要来自 self-correction 而非 library reuse。但接受 Rebuttal 关于 LLM-native DSL 设计（AIDL）的观点作为补充视角。
- **Gap 3（缝纫工艺语义缺失）**：维持"极高"评级。Rebuttal 提出的 FALCON 约束框架和 tool augmentation 方案在原理上有效，但从 combinatorial optimization 到连续几何+离散工艺的跨域迁移存在根本性挑战，不应低估。
- **Gap 4（跨系统互操作）**：维持"高"评级但补充新证据。接受 Rebuttal 关于 Textile IR 评价过于消极的批评，同时纳入 STEP-LLM（2026）等新工作作为佐证。

### 修订的结论
- **Gap 2（程序合成泛化能力）**：从"高"下调至"中-高"。接受 Rebuttal 关于"假二分法"的批评——分层架构（LLM 做高层决策 + solver 做精确计算）是一条可行的中间路线。但 SGP-Bench 的实际数据表明 LLM 对符号图形程序的语义理解远未达到 Rebuttal 所暗示的水平（最佳模型 SVG 理解准确率仅~67%，而非 >80%）。

### 新增的 Gap
- **Gap 5（Library Learning 的可靠性危机）**：基于新发现的反面证据新增。Rebuttal 将 LILO 式 library learning 作为解决 DSL 表达力瓶颈的核心方案，但 2024-2025 年的实证研究揭示了 LLM-based library learning 存在系统性失败模式。
- **Gap 6（LLM 几何推理能力的天花板）**：基于 SGP-Bench、AIDL、ShapeLib 等工作的实际数据新增。这些工作一致表明 LLM 在精确几何推理上存在根本性困难，直接影响程序合成路线的可行性。

### 方法论改进
- v0 过度聚焦于 GarmentCode 生态系统内部的局限；v1 将视野扩展到 program synthesis 领域的最新进展，同时对这些进展进行了更审慎的评估——既不像 v0 那样完全忽视，也不像 Rebuttal 那样过度乐观。
- 所有新增证据均来自 2024-2026 年的一手文献，经过交叉验证。

---

## Gap 1: DSL 表达力瓶颈

### 问题描述

当前所有基于程序合成的版型生成方法都依赖于 GarmentCode DSL 作为底层表示语言。该 DSL 的品类范围和几何表达能力存在根本性的上界限制：无法表示帽子、箱包、鞋类等非服装品类，对立体口袋、装饰性褶裥、拉链止口、特殊领型等复杂结构特征也缺乏原生支持。所有建立在 GarmentCode 之上的下游方法都不可能超越该 DSL 的表达边界。

Rebuttal 提出了三条反驳：(1) LILO 式 library learning 可自动扩展 DSL；(2) AIDL 式 LLM-native DSL 设计可从根源解决可用性问题；(3) Python 原生 API 可继承 LLM 的代码生成先验。这些反驳在理论方向上有价值，但实证基础存在显著缺陷。

### 对 Rebuttal 的回应与修正

**关于 Library Learning（LILO）**：Rebuttal 引用 LILO 的 REGEX domain 数据（93.2% 解决率 vs DreamCoder 45.6%）论证 DSL 可自动扩展。然而，两项独立的后续研究严重削弱了这一论证：

- **"Library Learning Doesn't"**（Berlot-Attwell et al., NeurIPS 2024）：对 LEGO-Prover 和 TroVE 两个 library learning 系统进行了深入分析，发现**函数复用极其罕见**。在 LEGO-Prover 中，20,000+ 已学习的 lemma 中仅有约 6% 被使用，且仅有 1 个 lemma 被复用过一次。TroVE 在 3,201 个测试问题中仅有 3 次成功复用。消融实验表明，性能提升来自 self-correction 和 self-consistency，而非 library reuse。
- **"LLM Library Learning Fails"**（Berlot-Attwell et al., 2025）：对 LEGO-Prover 的更深入分析，跨多个 LLM（GPT-4o-mini、GPT-4o、o3-mini）验证，发现"**LEGO-Prover does not outperform repeatedly prompting the model after accounting for computational budget**"。即：控制计算预算后，library learning 没有带来实际收益。

这些发现意味着：LILO 在 REGEX 和 LOGO 等简单领域的成功可能无法推广到服装版型这样的复杂几何领域。Library learning 的核心前提——系统能发现并有效复用抽象——在 LLM 时代尚未被可靠地实现。

**关于 LLM-Native DSL 设计（AIDL）**：Rebuttal 引用 AIDL 的 CLIP score（28.90 vs OpenSCAD 27.32）作为 LLM-native DSL 优越性的证据。这一方向在原理上有价值——为 LLM 重新设计 DSL 而非让 LLM 适应现有 DSL——但需要注意几个关键限制：

- AIDL 目前仅支持**2D CAD**，尚未验证在 3D 或复杂曲面建模中的适用性
- CLIP score 差异仅约 1.5 分，且 CLIP score 衡量的是视觉语义相似度，**不等同于几何精度**——服装版型需要毫米级精度
- AIDL 论文自身承认："LLM failed to generate syntactically correct programs in almost all cases" 当面对 FeatureScript 时——这说明 LLM 的 CAD 代码生成能力依然脆弱，即使换了 DSL 也可能面临类似挑战

**关于 Python 原生 API（Text-to-CadQuery）**：这一方向的实证支持最为充分。CADEvolve（2026）使用 CadQuery API 生成了约 8,000 个有效参数化 CAD 生成器，证明 Python-based CAD API + LLM 的组合在实践中可行。但 CADEvolve 也揭示了一个关键事实：即使使用 VLM 引导的进化方法，**单轮 VLM 生成仍然倾向于简单的 sketch-extrude 操作**，需要多代进化才能产生复杂结构。这说明 Python 原生 API 降低了门槛，但未消除表达力的核心挑战。

### 现有方法的局限（更新）

- **GarmentCode**（SIGGRAPH 2023）：DSL 设计范围明确排除非流形结构。仅覆盖上衣/裤装/裙装/连衣裙四大品类。
- **ChatGarment**（CVPR 2025）：创建 GarmentCodeRC 扩展版，但仍需"expanding garment type coverage"。扩展方式是逐个品类手工添加，不具备可扩展性。
- **Design2GarmentCode**（CVPR 2025）：完全依赖现有模板库，只能生成已有模板的参数变体。
- **AIpparel**（2025）："constrained to garments representable by manifold surfaces. Design elements like pockets require non-manifold structures."
- **AIDL**（Mistry et al., 2025）：提出 LLM-native DSL 设计原则（solver-aided + hierarchical + semantic），在 2D CAD 上展示了可行性，但**尚未在服装领域验证**，且依赖 constraint solver 的能力边界。
- **ShapeLib**（Jones et al., 2025）：展示了 LLM 引导的 3D 形状抽象库设计，但仅产生**立方体级别的部件包围盒**（cuboid primitives），远未达到服装版型所需的曲面精度。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| ChatGarment (Bian et al., CVPR 2025) | §Limitations: 需要 "expanding garment type coverage"；不支持 open-front jacket |
| AIpparel (2025) | §Limitations: "constrained to garments representable by manifold surfaces" |
| GarmentCode (Korosteleva & Sorkine-Hornung, 2023) | 仅覆盖四大品类，排除非流形结构 |
| AIDL (Mistry et al., 2025) | 提出 LLM-native DSL 设计，CLIP score 28.90 vs OpenSCAD 27.32，但仅限 2D CAD |
| Library Learning Doesn't (Berlot-Attwell et al., NeurIPS 2024) | Library reuse 极其罕见，性能提升来自 self-correction |
| LLM Library Learning Fails (Berlot-Attwell et al., 2025) | 控制计算预算后 library learning 无收益 |
| CADEvolve (Elistratov et al., 2026) | VLM + CadQuery 进化式生成 ~8k 参数化 CAD，但单轮 VLM 倾向简单结构 |
| ShapeLib (Jones et al., 2025) | LLM 引导库设计限于 cuboid primitives，未达曲面精度 |

### 影响程度评估

**极高（Critical）**——维持 v0 评级。Rebuttal 提出的三条路径（library learning、LLM-native DSL、Python API）在原理上指向正确方向，但当前的实证证据不足以将其从"基础性限制"降级为"工程挑战"。Library learning 的复用率问题尤为严重——如果系统无法有效复用自动发现的抽象，则自动扩展 DSL 的愿景无法实现。AIDL 式 LLM-native DSL 设计是最有前景的方向，但从 2D CAD 到服装版型的跨域迁移需要大量原创工作。

---

## Gap 2: 程序合成的泛化能力

### 问题描述

当前的程序合成方法本质上是**模板填参**（template-filling）——给定图像或文本，预测选择哪个 GarmentCode 模板及其参数值。v0 将此描述为"无法合成全新的版型程序"。Rebuttal 正确指出这是一个**假二分法**：在"trivial 参数替换"和"从零完全生成"之间存在丰富的中间地带。

### 对 Rebuttal 的回应与修正

**接受的批评**：

1. **分层架构的可行性**：AIDL 展示的"LLM 做高层语义结构 + constraint solver 做底层几何精度"的分工模式是一条务实的中间路线。这种架构不要求 LLM "从零合成"所有细节，而是让 LLM 负责结构性决策（几片、什么形状的连接关系），solver 处理精确计算。在服装版型领域，这意味着 LLM 决定版型拓扑，solver 计算曲线控制点和缝份——这比 v0 假设的"要么填参要么从零生成"更为现实。

2. **DreamCoder 引用过时**：接受批评。v0 使用 DreamCoder (2021) 作为 library learning 能力的参考，确实没有反映后续进展。但需要指出，即使是最新的 LILO (2024) 和 ShapeLib (2025) 也存在显著局限（见 Gap 5）。

3. **Tool augmentation 的价值**：CAD-Assistant (Sarda et al., 2024) 展示了 tool augmentation 可在不重新训练的情况下扩展 LLM 的操作空间，constraint checker 单独将 PF1 从 0.747 提升到 0.979。这一思路对服装版型有直接参考价值。

**需要修正的过度乐观**：

1. **SGP-Bench 的实际数据与 Rebuttal 声称不符**：Rebuttal 声称"LLM 在坐标扰动下展现出 >80% 的一致性"。查阅 SGP-Bench 原文数据后发现，这里的"一致性"（consistency）指的是**同一模型在坐标变换前后给出相同答案的比例**，而非**答案的正确率**。实际上：
   - 最佳开源模型（Mistral-Large2-123B）SVG 理解准确率仅 57.2%
   - 最佳模型（Claude 3.5 Sonnet）SVG 理解准确率约 67%
   - CAD 理解最佳约 74%
   - 高一致性 + 低准确率 = LLM 在坐标变换下**一致地给出错误答案**
   - SGP-Bench 论文自身指出："open-source models struggle with 'semantic' questions"，"the symbolic MNIST dataset... GPT-4o can only achieve a chance-level performance"

   这意味着 LLM 对符号程序的理解能力远未达到可靠生成服装版型程序的水平。

2. **FALCON 的跨域迁移问题**：FALCON 在 TSP 等 combinatorial optimization 问题上达到 100% 可行性，但这些是**离散优化问题**，其约束可以精确定义和验证。服装版型涉及**连续几何约束**（曲线平滑性、曲面可展开性）和**物理约束**（面料悬垂性、缝合可行性），这些约束难以形式化为 grammar rule 或自动修复目标。从 CO 到 garment pattern 的迁移距离被 Rebuttal 低估了。

### 现有方法的局限（更新）

- **Design2GarmentCode**（CVPR 2025）："currently cannot substantially alter GarmentCode's underlying structure and logistics"
- **ChatGarment**（CVPR 2025）：LLM 微调学习 DSL 语法，但只能学会已有程序结构的分布
- **AIDL**（Mistry et al., 2025）：分层架构（LLM + solver）在 2D CAD 上可行，CLIP score 优于 OpenSCAD。但服装版型的约束复杂度远超 2D 几何拼接
- **CAD-Assistant**（Sarda et al., 2024）：tool augmentation 可扩展泛化能力，constraint checker 效果显著。但在服装领域缺乏对应的"缝制可行性检查器"
- **CADEvolve**（2026）：进化式方法可生成超越单轮 VLM 能力的复杂 CAD 程序，但需要数千代进化，效率存疑

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| Design2GarmentCode (Zhou et al., CVPR 2025) | 无法改变 GarmentCode 底层结构 |
| SGP-Bench (Qiu et al., 2024) | 最佳 LLM SVG 理解准确率仅~67%，semantic 问题准确率更低 |
| AIDL (Mistry et al., 2025) | 分层架构 LLM+solver 在 2D CAD 可行，但 LLM 仍无法处理 FeatureScript |
| CAD-Assistant (Sarda et al., 2024) | Tool augmentation 有效，constraint checker PF1 0.747→0.979 |
| FALCON (Liu et al., 2026) | 100% 可行性仅限 combinatorial optimization，非连续几何问题 |
| CADEvolve (Elistratov et al., 2026) | 进化式 CAD 生成有效但需多代迭代 |

### 影响程度评估

**中-高（Medium-High）**——从 v0 的"高"略微下调。接受 Rebuttal 关于分层架构和 tool augmentation 的论点：泛化能力的提升不一定需要 LLM 自主创造全新结构，而可以通过 LLM 做结构决策 + solver/tools 做精确计算的分工来实现。但 SGP-Bench 的实际数据表明 LLM 的符号图形理解能力仍存在显著差距，且从 2D CAD 到服装版型的跨域迁移尚无验证。短期可解性从"中"维持不变——分层架构是正确方向，但构建服装领域的 constraint solver 和可行性检查器本身就是重大研究挑战。

---

## Gap 3: 缝纫工艺语义的缺失

### 问题描述

当前所有版型生成方法都将"版型"定义为几何形状的集合（二维面片 + 缝合边对应关系），完全忽略了对生产至关重要的工艺语义信息：缝合工艺类型、缝合顺序约束、缝份方向与处理、辅料标注、裁剪与排料规则。

### 对 Rebuttal 的回应与修正

**部分接受的论点**：

1. **FALCON 三层架构的类比价值**：Grammar-constrained decoding → feasibility repair → adaptive sampling 的分层保证框架在概念上可迁移到缝纫约束。特别是第二层（feasibility repair）的思路——生成后修复语义违规——对于缝合边长度不匹配、缝份宽度不符标准等问题确实适用。

2. **AIDL 约束系统的类比**：Coincident ↔ notch matching、Equal ↔ seam allowance matching 等映射关系在概念层面成立。"缝纫语义可以编码为约束，由 constraint solver 处理"这一思路在原理上正确。

3. **Tool augmentation 的务实性**：缝份计算器、notch 生成器、grain line 检查器等工具化方案是工程上最可行的短期路径。

**需要修正的过度简化**：

1. **"DAG 结构化知识并非未解决的问题"的误判**：Rebuttal 认为工序排序"在工业工程领域已有数十年的成熟方法"。这在一般制造业中正确，但**服装缝制的工序 DAG 具有独特的复杂性**：

   - 服装工序约束不仅是硬约束（"必须先缝肩缝再装领子"），还涉及**质量相关的软约束**（"先做省道再拼接会减少面料扭曲"）和**设备相关的约束**（"包缝机和平缝机的切换需要考虑效率"）
   - 真实的服装工序 DAG 包含**数百个节点**（一件复杂西装可能有 200+ 个缝制步骤），且不同设计变体的 DAG 拓扑结构完全不同
   - 关键不是"能否对工序 DAG 建模"（确实有成熟方法），而是**如何从版型几何自动推导工序 DAG**——这需要将几何信息、材料属性、缝合类型和制造约束综合起来，目前没有任何方法能做到这一点

2. **FALCON 的跨域迁移挑战**：FALCON 在 CO 问题上的 100% 可行性基于两个前提：(a) 约束可以精确形式化为 grammar rule；(b) 修复操作有明确定义。服装缝纫的约束多为**隐性知识**（experienced sewers 的 tacit knowledge），难以完全形式化。例如，"法式缝适合轻薄面料的弧形缝线"这类知识需要面料力学和缝制工艺的交叉理解。

3. **"不需要把所有工艺知识编码进 DSL"的前提有问题**：Tool augmentation 的前提是 LLM 知道"何时调用什么工具"。但对于缝纫工艺，LLM 首先需要理解**设计意图与工艺选择之间的关系**（为什么这条缝用包缝而不是法式缝？因为它在内部且需要耐久性），这种推理能力目前没有被任何工作验证过。

### 现有方法的局限（更新）

- **GarmentCode 系列**：DSL 中无缝合工艺字段，输出仅含几何映射
- **DressCode / SewingGPT**（SIGGRAPH 2024）：序列表示不含工艺信息
- **SewFormer**（CVPR 2023）：预测面片参数和缝合关系，"缝合"仅指几何对应
- **FALCON**（Liu et al., 2026）：三层可行性保证框架概念可迁移，但需要将离散 CO 约束重新定义为连续几何+离散工艺的混合约束体系
- **AIDL**（Mistry et al., 2025）：约束系统在 2D 几何关系上有效，但缺乏面料属性、工艺类型等服装特定约束

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| fashionista.ai 行业报告 | AI 需要人工干预处理 notch 和 seam allowance |
| GarmentCode (2023) | DSL 定义中无缝合工艺字段 |
| DressCode/SewingGPT (2024) | 序列表示不含工艺信息 |
| FALCON (Liu et al., 2026) | 三层可行性框架在 CO 问题上达 100%，但约束类型与服装不同 |
| AIDL (Mistry et al., 2025) | 几何约束系统有效但不含面料/工艺维度 |
| 工业版型实践 | 版型标注包含缝份、布纹线、剪口位等数十种工艺符号，均未被学术方法捕获 |

### 影响程度评估

**极高（Critical）**——维持 v0 评级。Rebuttal 提出的 tool augmentation 路径在工程上可行，是最有希望的短期方案。但"缝纫工艺语义"的核心挑战不在于"能否对已知约束建模"（可以），而在于：(1) 如何获取和形式化大量隐性工艺知识；(2) 如何从版型几何自动推导制造约束；(3) 如何处理面料-工艺-设计三者之间的复杂耦合关系。这些仍是开放性研究问题。

---

## Gap 4: 跨 DSL / 跨 CAD 系统的互操作

### 问题描述

学术版型生成方法与工业 CAD 系统（CLO3D、Style3D、Optitex、Lectra Modaris、Gerber AccuMark）之间存在巨大的格式鸿沟。学术输出格式仅含裸几何信息，工业标准格式（DXF/AAMA、ASTM）包含丰富的元数据。

### 对 Rebuttal 的回应与修正

**接受的批评**：

1. **对 Textile IR 的评价过于消极**：v0 将其描述为"仅仅是概念性提案"确实不够严谨。Textile IR（Teikari & Fuenmayor, 2026）包含了正式的节点类型规范（PatternPiece, SeamEdge, Dart, Notch, MaterialRegion）、双向流转示例和复合不确定性分析。将其与 LLVM IR 的发展历程对比，Textile IR 处于合理的早期阶段。v1 修正为：Textile IR 提供了有价值的理论框架，但尚需社区采用和实现验证。

2. **LLM 直接生成工业格式的可行性**：Text-to-CadQuery 和 CAD-LLaMA 证明了 LLM 可直接调用 CAD Python API，99.90% 语法成功率。DXF 的核心实体（LINE、ARC、POLYLINE）在 Python 中有成熟库支持（ezdxf）。从技术角度看，GarmentCode → DXF 的单向转换在工程上并非不可行。

3. **STEP-LLM（2026）的新证据**：Shi et al. 提出直接使用 LLM 生成 STEP 格式的 CAD 模型，实体计数的平均值和分布更接近 ground truth。STEP 是工业级 3D CAD 的标准交换格式，这一工作说明 LLM 直接生成工业格式不再是空想。

**仍需保留的关切**：

1. **语法正确不等于工业可用**：99.90% 的语法成功率令人印象深刻，但工业版型的互操作不仅是格式问题——还涉及**语义互操作**：尺码规则（grading rules）、缝份分配（seam allowance assignment）、布纹方向（grain line）、放码点（grade points）等元数据的正确传递。这些在 DXF 中表示为图层和自定义属性，需要领域知识才能正确生成。

2. **双向互操作远未解决**：当前讨论主要集中在学术→工业的单向转换。但真实工作流中还需要工业→学术的反向流（例如，导入已有 CLO3D 版型进行 AI 修改后再导出），这要求 AI 系统能**解析**工业格式的全部语义，远比生成困难。

### 现有方法的局限（更新）

- **GarmentCode 系列**：输出为 Python 程序 + SVG/JSON 中间格式，无工业 CAD 互操作
- **DressCode / SewingGPT**：自定义 JSON 格式
- **Textile IR**（Teikari & Fuenmayor, 2026）：提供了有价值的理论框架和节点类型规范，但尚无可用的实现代码
- **Text-to-CadQuery / CAD-LLaMA**：证明 LLM 可直接生成 CAD API 代码，99.90% 语法成功率
- **STEP-LLM**（Shi et al., 2026）：LLM 直接生成 STEP 格式，实体计数接近 ground truth
- **Valentina / Seamly2D**：开源参数化系统与 GarmentCode 逻辑不同，无桥接方案

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| Design2GarmentCode (CVPR 2025) | 提及工业 CAD 系统但不提供互操作 |
| Textile IR (Teikari & Fuenmayor, 2026) | 正式节点类型规范，但无实现代码 |
| Text-to-CadQuery (Kim et al., 2025) | 124M 模型超越 363M Text2CAD，Python API 路线可行 |
| STEP-LLM (Shi et al., 2026) | LLM 直接生成 STEP 格式，实体计数接近 ground truth |
| CAD-LLaMA | SPCC 达 99.90% 语法成功率 |
| 工业实践 | DXF/ASTM 格式包含尺码规则、放码点等学术格式不含的元数据 |

### 影响程度评估

**高（High）**——维持 v0 评级，但短期可解性从"中"上调至"中-高"。新证据（STEP-LLM、CAD-LLaMA 的高语法成功率）表明学术→工业的单向格式转换在技术上比 v0 评估时更加可行。最有希望的短期方案是：(1) 开发 GarmentCode → DXF 单向转换器，使用 ezdxf 库；(2) 利用 LLM 直接生成包含工业元数据的版型代码。但双向互操作和完整语义保持仍是中期挑战。

---

## Gap 5: Library Learning 的可靠性危机（新增）

### 问题描述

Rebuttal 将 LILO 式 library learning 作为解决 DSL 表达力瓶颈（Gap 1）和泛化能力不足（Gap 2）的核心方案。然而，2024-2025 年的系统性实证研究揭示了 LLM-based library learning 存在根本性的可靠性问题：自动发现的库函数**在实践中几乎不被复用**，而报告的性能提升主要来自其他机制（self-correction、self-consistency、increased sampling）。这意味着将 library learning 作为服装版型 DSL 自动扩展的基础方案存在重大风险。

### 核心证据

**1. "Library Learning Doesn't"（Berlot-Attwell et al., NeurIPS 2024）**

对 LEGO-Prover（formal theorem proving）和 TroVE（programmatic math reasoning）两个 library learning 系统的深入分析：

- **LEGO-Prover**：系统学习了 20,000+ lemma，但在最终求解步骤中仅使用了约 1,233 个（~6%）。在这些被使用的 lemma 中，**仅有 1 个被跨问题复用过**，且仅复用了一次。消融实验（移除跨问题共享）几乎不影响性能——"ablation's performance is strong, solving only 1 question less than the baseline"。
- **TroVE**：最终库仅包含 15 个函数，其中仅 2 个被成功复用（共 3 次，在 3,201 个测试问题中）。消融实验（禁用 Import mode）在大多数子集上**性能反而更好**。

论文结论："both TroVE and LEGO-Prover do not directly reuse the tools they learn... self-correction and self-consistency are the primary drivers of the observed performance gains."

**2. "LLM Library Learning Fails"（Berlot-Attwell et al., 2025）**

更深入的后续研究，使用多个 LLM（GPT-4o-mini、GPT-4o、o3-mini）重新评估 LEGO-Prover：

- 在控制计算预算的条件下，LEGO-Prover **不优于简单的重复提示基线**（Draft-Sketch-Prove）
- "We find no evidence of the direct reuse of learned lemmas, and find evidence against the soft reuse of learned lemmas"
- 关键发现：library learning 的表观性能提升是因为系统进行了更多次尝试（类似于 best-of-N sampling），而非因为学到了有用的可复用工具

**3. ShapeLib 的间接佐证（Jones et al., 2025）**

ShapeLib 在评论 LILO 时指出："in LILO, the LLM is only used to *name* functions found by a non-semantic compression-based method, so the prior issues still remain." 即 LILO 的 LLM 组件主要贡献是命名和文档化，而非发现语义上有意义的抽象。

**4. 对 Rebuttal 引用数据的修正**

Rebuttal 声称 LILO "在 REGEX domain 达到 93.2% 解决率"且"offline synthesis 性能比基础 DSL 高出 42.27 分"。这些数据来自 LILO 论文本身，但需要注意：
- REGEX 是一个相对简单的符号处理域，其抽象模式（如"元音字母"= a|e|i|o|u）非常规则且可组合。服装版型的几何结构远为复杂，涉及连续参数、空间关系和物理约束。
- LILO 的 AutoDoc 实验显示**有命名的抽象比匿名抽象好得多**（+9.73 分 vs -30.60 分），这一发现有价值。但它也暗示了一个问题：如果命名不当或语义标签与实际功能不匹配（在服装等复杂领域更容易发生），library learning 反而会损害性能。

### 影响程度评估

**高（High）**。Library learning 被 Rebuttal 定位为解决 Gap 1 和 Gap 2 的关键技术路线。如果这条路线不可靠，则 Rebuttal 对 Gap 1-2 严重性的降级建议失去了核心支撑。这并不意味着 library learning 的研究方向无价值——ShapeLib 展示了一种更有前景的方法（人类提供设计意图 + LLM 引导实现 + 几何验证），但这种方法需要大量领域专家参与，不是 Rebuttal 所描述的"自动扩展"。

对于 FABRIC-FRONTIER 项目，这意味着：**不应将自动 library learning 作为技术路线的基石**。更可靠的方案是：(1) 人工设计核心版型原语，由领域专家把关质量；(2) 使用 LLM 辅助发现候选抽象，但由人工验证和筛选；(3) 参考 ShapeLib 的混合方法，将 LLM 的语义先验与几何验证相结合。

---

## Gap 6: LLM 几何推理能力的天花板（新增）

### 问题描述

程序合成路线的隐含前提是 LLM 具有足够的几何推理能力来生成正确的版型程序。然而，2024-2025 年多项基准测试和系统工程实践一致表明，**LLM 在精确几何推理上存在根本性困难**。这一能力差距不是简单的"工程挑战"，而是影响整条程序合成路线可行性的基础性限制。

### 核心证据

**1. SGP-Bench 的量化数据（Qiu et al., 2024）**

对 20+ 个 LLM 在符号图形程序理解上的系统性基准测试：

- 所有模型平均准确率：SVG < 70%，CAD < 75%
- 最佳模型（Claude 3.5 Sonnet）：SVG 约 67%，CAD 约 74%
- **语义理解**准确率（"这个 SVG 画的是什么"）远低于**颜色理解**（"哪个部分是红色"）——最佳开源模型在语义问题上仅 37.6%，颜色上 81.6%
- 关键发现：GPT-4o 在 symbolic MNIST 数据集上**仅达到随机水平**——即 LLM 无法从 SVG 代码判断画的是数字几
- 论文结论："LLMs still struggle to understand complex geometric layouts and often misinterpret or misattribute constraints and relations between parametric controls"（引用自 ShapeLib）

**2. AIDL 的设计动机本身就是证据（Mistry et al., 2025）**

AIDL 论文的核心动机是：LLM 在使用传统 CAD 语言时表现很差。具体地：
- 当 LLM 尝试使用 FeatureScript（工业 CAD 语言）时，"LLM failed to generate syntactically correct programs in **almost all cases**"
- AIDL 的解决方案是为 LLM 重新设计语言——这本身就承认了 LLM 的几何推理能力不足
- 即使使用 AIDL，结果仅通过 CLIP score 评估（视觉语义相似度），**不保证几何精度**

**3. CADEvolve 的实践经验（Elistratov et al., 2026）**

- "single-pass VLMs struggle to reconstruct industrial-grade CAD programs: they tend to saturate on extruded prisms and fail to chain heterogeneous operations reliably"
- 需要**多代进化**（propose-execute-filter loop）才能产生复杂结构
- 即使经过进化，最终产出的 ~8k 参数化生成器中仍主要是相对简单的机械零件，远未达到服装版型所需的复杂曲面操作

**4. SVGenius 基准的补充数据（Chen et al., 2025）**

- 评估 22 个主流模型在 SVG 理解、编辑和生成上的表现
- "all models exhibit systematic performance degradation with increasing complexity"
- "style transfer remains the most challenging capability across all model types"
- 服装版型涉及的几何复杂度远超 SVGenius 的测试范围

### 对服装版型的特殊影响

服装版型的几何推理比通用 CAD 更具挑战性：

1. **曲线主导**：服装版型的轮廓线以贝塞尔曲线和样条曲线为主（领口弧线、袖山弧线、下摆曲线），而非 CAD 常见的直线-圆弧组合。LLM 对曲线控制点的推理能力几乎未被测试。

2. **2D→3D 的心智模型**：版型师需要在 2D 面片和 3D 成衣效果之间来回推理——一条省道线的位置和深度如何影响 3D 合体效果？这种 2D↔3D 心智旋转能力是服装版型的核心，但对 LLM 来说是极端困难的空间推理任务。

3. **累积误差效应**：一件衣服由 10-30 个面片组成，每个面片的轮廓误差会在缝合时累积。如果 LLM 在单个面片上的几何精度为 95%，那么 20 个面片全部正确的概率仅为 0.95^20 ≈ 36%。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| SGP-Bench (Qiu et al., 2024) | 最佳 LLM SVG 理解~67%；GPT-4o symbolic MNIST 仅随机水平 |
| SVGenius (Chen et al., 2025) | 所有模型在复杂度增加时系统性退化 |
| AIDL (Mistry et al., 2025) | LLM 无法使用 FeatureScript 生成语法正确的 CAD 程序 |
| CADEvolve (Elistratov et al., 2026) | 单轮 VLM 无法生成工业级 CAD，倾向简单结构 |
| ShapeLib (Jones et al., 2025) | "LLMs still struggle to understand complex geometric layouts" |
| LLMs for CAD Survey (Zhang et al., 2025) | 系统性综述 LLM-CAD 集成的局限 |

### 影响程度评估

**高（High）**。LLM 几何推理能力的天花板是整条程序合成路线的隐性风险——如果 LLM 无法可靠地理解和生成几何程序，则无论 DSL 如何设计、library 如何学习，最终输出的版型质量都会受到根本限制。AIDL 的 solver-aided 方法（将精确计算交给 solver）和 CADEvolve 的进化方法（通过多轮迭代筛选）都是绕过这一限制的有效策略，但它们增加了系统复杂度且引入了新的挑战（solver 设计、评估函数设计）。

对于 FABRIC-FRONTIER 项目的启示：**不应期望 LLM 直接输出高精度版型程序**。更可靠的架构是将 LLM 定位为"结构级决策者"（决定版型拓扑和大致比例），而将精确几何计算委托给确定性的 constraint solver 或参数化建模引擎。

---

## 总结：方向一研究差距全景（v1）

| Gap | 核心问题 | v0 评级 | v1 评级 | 变更原因 | 短期可解性 |
|-----|----------|---------|---------|----------|------------|
| Gap 1: DSL 表达力瓶颈 | 品类和特征覆盖不足 | 极高 | **极高** | Library learning 实证不支持自动扩展；AIDL 方向正确但未在服装领域验证 | 低 |
| Gap 2: 程序合成泛化能力 | 只能填参，不能创造新程序 | 高 | **中-高** | 接受分层架构（LLM+solver）作为可行中间路线 | 中 |
| Gap 3: 缝纫工艺语义缺失 | 只有几何，没有工艺 | 极高 | **极高** | Tool augmentation 方向正确，但工艺知识的形式化和自动推导仍是开放问题 | 低 |
| Gap 4: 跨系统互操作 | 学术与工业格式隔离 | 高 | **高** | 新证据（STEP-LLM、CAD-LLaMA）支持单向转换可行性 | 中-高 |
| Gap 5: Library Learning 可靠性（新） | 自动库学习复用率极低 | — | **高** | 两篇独立研究证明 LLM library learning 的核心前提不成立 | 低 |
| Gap 6: LLM 几何推理天花板（新） | LLM 精确几何推理能力不足 | — | **高** | 多项基准测试一致显示 LLM 在复杂几何推理上存在系统性缺陷 | 低-中 |

### 核心洞察（v1 更新）

v0 的核心洞察——"程序合成路线的四个差距相互关联"——仍然成立，但需要补充两个新的维度：

1. **技术路线的可靠性基础**：Rebuttal 提出的解决方案（library learning、FALCON 约束、tool augmentation）在理论上可行，但 Gap 5（library learning 的复用率问题）和 Gap 6（LLM 几何推理天花板）表明这些方案的实际效果可能被高估。**解决方案本身也存在未经验证的假设。**

2. **领域特殊性**：Rebuttal 大量引用通用 program synthesis / CAD 领域的进展（LILO 在 REGEX、FALCON 在 TSP、AIDL 在 2D CAD），但服装版型具有独特的领域特征——曲线主导的几何、2D↔3D 心智旋转、面料物理属性、缝纫工艺约束——这些特征使得跨域迁移的难度被系统性低估。

3. **推荐的架构方向**：综合所有证据，最有前景的架构是 **AIDL 启发的分层系统**：
   - **LLM 层**：负责高层结构决策（版型拓扑、部件关系、设计意图解读）
   - **Constraint Solver 层**：负责精确几何计算（曲线拟合、缝份计算、ease 分配）
   - **Tool 层**：封装领域工艺知识（缝合类型选择、工序排序、面料兼容性检查）
   - **格式转换层**：使用 ezdxf 等工具桥接学术与工业格式

   这种架构利用 LLM 的语义理解优势，同时通过 solver 和 tools 弥补其精确推理的不足。但每一层的构建都需要大量的领域特定工作——这不是一个短期可解的"工程问题"，而是需要服装工程与 AI 交叉研究的长期项目。
