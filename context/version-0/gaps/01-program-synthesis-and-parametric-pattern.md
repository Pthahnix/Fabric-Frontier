<!-- markdownlint-disable -->
# 方向一：文本/图像 → 参数化版型程序合成

## 概述

程序合成（Program Synthesis）路线是当前 AI 版型生成领域中最具学术吸引力的方向之一。其核心思想是：将服装版型建模为一段可执行程序（通常基于领域特定语言 DSL），然后利用大语言模型或专用神经网络从文本描述或图像输入合成该程序。以 GarmentCode（Korosteleva & Sorkine-Hornung, SIGGRAPH 2023）为代表的参数化 DSL 奠定了这一范式的基础，后续的 Design2GarmentCode（Zhou et al., CVPR 2025）和 ChatGarment（Bian et al., CVPR 2025）在此基础上分别探索了视觉输入到程序的映射和交互式对话生成。然而，这条技术路线存在多个根本性缺陷，严重限制了其向工业级应用的演进。

---

## Gap 1: DSL 表达力瓶颈

### 问题描述

当前所有基于程序合成的版型生成方法都依赖于 GarmentCode DSL 作为底层表示语言。然而，GarmentCode 的设计初衷是覆盖常见服装品类的参数化建模，其品类范围和几何表达能力存在根本性的上界限制。该 DSL 无法表示帽子、箱包、鞋类、配饰等非服装品类，对于服装内部的复杂结构特征——如立体口袋、装饰性褶裥、拉链止口、特殊领型（如青果领、荷叶领）——也缺乏原生支持。所有建立在 GarmentCode 之上的下游方法（Design2GarmentCode、ChatGarment、AIpparel）都不可能超越该 DSL 的表达边界，形成了一个"天花板效应"：**无论神经网络多么强大，其输出空间被 DSL 的词汇表和语法规则硬性限定。**

### 现有方法的局限

- **GarmentCode**（SIGGRAPH 2023）：定义了一套基于 Python 的参数化版型 DSL，支持上衣、裤装、裙装、连衣裙等基本品类。但其设计范围明确排除了非流形结构（non-manifold structures），即任何需要面料重叠、多层拼接或立体结构的设计元素。
- **ChatGarment**（CVPR 2025）：为了扩展 GarmentCode 的覆盖范围，作者专门创建了 "GarmentCodeRC"（Refined & Comprehensive）版本，增加了更多版型模板和参数。但论文中明确承认仍需 "expanding garment type coverage"，且无法支持开襟夹克（open-front jacket）等常见款式，除非额外添加专用扩展模块。这说明 DSL 的扩展方式是**逐个品类手工添加**，而非系统性的表达能力增长。
- **Design2GarmentCode**（CVPR 2025）：完全依赖 GarmentCode 的现有模板库，只能生成已有模板的参数变体。
- **AIpparel**（2025）：在其局限性章节中明确声明："constrained to garments representable by manifold surfaces. Design elements like pockets require non-manifold structures."（受限于可由流形曲面表示的服装。口袋等设计元素需要非流形结构。）

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| ChatGarment (Bian et al., CVPR 2025) | 论文 §Limitations 明确提出需要 "expanding garment type coverage"；不支持 open-front jacket |
| ChatGarment (Bian et al., CVPR 2025) | 创建 GarmentCodeRC 扩展版，说明原始 DSL 覆盖不足 |
| AIpparel (2025) | §Limitations: "constrained to garments representable by manifold surfaces" |
| GarmentCode (Korosteleva & Sorkine-Hornung, 2023) | 仅覆盖上衣/裤装/裙装/连衣裙四大品类 |

### 影响程度评估

**极高（Critical）**。DSL 表达力是整条程序合成路线的根基。如果 DSL 无法表示目标服装，则所有下游方法——无论采用何种神经网络架构、训练多少数据——都无法生成该服装的版型。当前的扩展方式（手工添加模板）不具备可扩展性，每增加一个新品类都需要领域专家的大量手工编程工作。这一瓶颈将程序合成路线的应用范围锁定在了"学术演示"级别，距离覆盖工业版型设计的全品类需求有巨大鸿沟。

---

## Gap 2: 程序合成的泛化能力

### 问题描述

当前的程序合成方法本质上是**模板填参**（template-filling）而非**真正的程序合成**（program synthesis）。Design2GarmentCode 和 ChatGarment 的工作方式是：给定一张图像或一段文本，预测应该选择哪个 GarmentCode 模板，以及该模板的参数值应该是多少。它们无法合成全新的版型程序——即创造从未见过的面片拓扑结构（panel topology）、发明新的构造逻辑、或组合已有构造单元形成新的设计模式。这就好比一个"程序合成器"只能填写已有函数的参数，却无法编写新函数。

### 现有方法的局限

- **Design2GarmentCode**（CVPR 2025）：论文结论部分明确承认："currently cannot substantially alter GarmentCode's underlying structure and logistics, which impacts generation quality due to inherent limitations in GarmentCode's design and modeling capabilities"（目前无法实质性地改变 GarmentCode 的底层结构和逻辑，这由于 GarmentCode 设计和建模能力的固有局限而影响了生成质量）。尽管论文 Figure 9 展示了生成 "layered-skirt component" 作为新程序的案例，但该示例仍处于 GarmentCode 的现有范式之内，并非拓扑级别的创新。
- **ChatGarment**（CVPR 2025）：通过 LLM 微调学习 DSL 语法，但 LLM 只能学会已有程序结构的分布，无法从零构建新的构造逻辑。这与传统的代码补全本质相同——可以生成符合语法的代码，但无法发明新的算法。
- **对比参照——DreamCoder**（Ellis et al., 2021）：在通用程序合成领域，DreamCoder 展示了通过归纳程序合成（inductive program synthesis）自动扩展 DSL 库的能力——系统在解决任务的过程中发现可复用的子程序，并将其添加到 DSL 中，从而不断扩展自身的表达能力。这种 "library learning" 机制在版型生成领域**完全没有对应的工作**。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| Design2GarmentCode (Zhou et al., CVPR 2025) | 论文 §Conclusion 明确承认无法改变 GarmentCode 底层结构 |
| Design2GarmentCode (Zhou et al., CVPR 2025) | Figure 9 的 "新程序" 仍在 GarmentCode 范式内 |
| DreamCoder (Ellis et al., 2021) | 展示了 DSL library learning 能力，但未应用于服装领域 |
| ChatGarment (Bian et al., CVPR 2025) | LLM 微调仅学习语法分布，无法创造新结构 |

### 影响程度评估

**高（High）**。泛化能力的缺失意味着程序合成方法的创新空间被严格限制。在时尚设计中，创新的核心往往不是已有款式的参数微调，而是结构层面的重新组合与发明（如解构主义设计、不对称裁剪、立体剪裁）。当前方法无法支持这类设计探索，将 AI 版型生成锁定在了"已知服装的参数变体"这一狭窄区间内。要突破这一瓶颈，需要借鉴 DreamCoder 等工作的思路，构建能够自主发现和扩展构造原语的程序合成系统。

---

## Gap 3: 缝纫工艺语义的缺失

### 问题描述

当前所有版型生成方法——无论是程序合成、扩散模型还是自回归模型——都将"版型"定义为**几何形状的集合**（一组二维面片 + 缝合边对应关系）。然而，真实的服装版型还包含大量**工艺语义信息**，这些信息对于将版型转化为可制造的成衣至关重要，却被现有研究完全忽略：

1. **缝合工艺类型**：包缝（overlock）、平缝（flat seam）、法式缝（French seam）、来去缝（fell seam）等，不同工艺影响缝份宽度、外观和耐久性
2. **缝合顺序约束**：必须先缝肩缝再装领子，先合侧缝再做下摆，先做省道再拼接面片——这些顺序规则直接影响成衣质量
3. **缝份方向与处理**：缝份倒向（pressing direction）、缝份劈开（open seam）或倒向一侧（closed seam）
4. **辅料标注**：拉链位置与类型、纽扣间距、衬布区域、弹力带走向
5. **裁剪与排料规则**：布纹方向（grain line）、对条对格要求、面料利用率优化

### 现有方法的局限

- **GarmentCode 系列**：DSL 中没有任何字段用于描述缝合工艺类型或缝合顺序。输出的"缝合信息"仅是"边 A 与边 B 对应缝合"的几何映射，完全缺乏工艺维度。
- **DressCode / SewingGPT**（SIGGRAPH 2024）：序列化表示中包含面片轮廓和缝合边配对，但同样不含任何工艺信息。
- **SewFormer**（CVPR 2023）：预测面片参数和缝合关系，但"缝合"仅指几何对应，不涉及工艺选择。
- **工业实践对比**：在真实的版型制作流程中，版型师需要标注数十种工艺符号（缝份宽度、止口线、剪口位、打孔位、布纹线等）。fashionista.ai 的行业报告明确指出，AI 系统在缝口（notch）和缝份余量（seam allowance）方面需要人工干预，因为这些信息超出了当前 AI 的输出范围。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| fashionista.ai 行业报告 | 明确指出 AI 需要人工干预才能处理 notch 和 seam allowance |
| GarmentCode (2023) | DSL 定义中无缝合工艺字段 |
| DressCode/SewingGPT (2024) | 序列表示不含工艺信息 |
| 工业版型实践 | 版型标注包含缝份、布纹线、剪口位等数十种工艺符号，均未被任何学术方法捕获 |

### 影响程度评估

**极高（Critical）**。缝纫工艺语义的缺失是 AI 版型生成从"学术演示"走向"工业可用"的最大障碍之一。没有工艺信息的版型本质上是一张"几何草图"，无法直接用于生产。这意味着即使 AI 生成了几何上完美的面片形状，仍然需要专业版型师补充全部工艺细节——而这些细节的工作量可能占到整个版型制作的 30-50%。更根本的问题是：**缝合顺序约束构成了一个有向无环图（DAG），这种结构化知识在当前任何版型 DSL 或神经网络架构中都没有被建模。** 这是一个全新的研究方向，需要将制造工艺知识与几何生成深度整合。

---

## Gap 4: 跨 DSL / 跨 CAD 系统的互操作

### 问题描述

当前所有学术版型生成方法都生活在一个封闭的生态系统中：GarmentCode 输出 GarmentCode 格式，SewingGPT 输出自定义 JSON，GarmentDiffusion 输出面片坐标序列。这些格式与工业界使用的 CAD 系统（CLO3D、Style3D、Optitex、Lectra Modaris、Gerber AccuMark）之间存在巨大的格式鸿沟。工业标准格式（DXF/AAMA、ASTM）包含丰富的元数据（布纹方向、缝份、标注符号、尺码规则），而学术输出格式仅包含裸几何信息。这种互操作性的缺失使得学术研究成果无法被工业界直接采用。

### 现有方法的局限

- **GarmentCode 系列**：输出为 Python 程序 + SVG/JSON 中间格式，无法直接导入任何工业 CAD 系统。Design2GarmentCode 论文虽然在引言中提及 Assyst、Modaris、Optitex 等工业工具作为背景，但自身**完全不提供**与这些系统的互操作能力。
- **DressCode / SewingGPT**：自定义 JSON 格式，与工业标准无关。
- **开源参数化版型系统**：Valentina 和 Seamly2D 是工业级开源参数化版型软件，但其参数化模型的设计逻辑与 GarmentCode 完全不同（Valentina 基于量体数据驱动的参数化公式，GarmentCode 基于程序化几何构造）。两者之间**没有任何转换工具或桥接方案**。
- **Textile IR 提案**（Teikari & Fuenmayor, 2026）：提出了一种双向中间表示（bidirectional intermediate representation），旨在桥接不同的版型表示格式。但截至目前，这仅是一个概念性提案，**没有任何可用的实现代码或验证实验**。

### 证据来源

| 来源 | 关键证据 |
|------|----------|
| Design2GarmentCode (Zhou et al., CVPR 2025) | 引言提及工业 CAD 系统但不提供互操作 |
| Textile IR (Teikari & Fuenmayor, 2026) | 提出双向 IR 概念但无实现 |
| 工业实践 | DXF/ASTM 格式包含缝份、布纹线、标注等元数据，学术格式均不包含 |
| Valentina / Seamly2D | 开源参数化系统与 GarmentCode 逻辑完全不同，无桥接方案 |

### 影响程度评估

**高（High）**。互操作性是技术转化的关键瓶颈。当前的状况意味着：学术界在 GarmentCode 生态内不断迭代优化，而工业界继续使用完全不同的工具链，两个世界之间**没有桥梁**。即使学术方法在生成质量上取得突破，如果输出无法导入 CLO3D 或 Lectra，对于服装企业来说就等于不存在。Textile IR 的提案方向是正确的，但从提案到可用的工业级工具仍有巨大的工程量。一个可能的中间方案是：开发 GarmentCode → DXF 的单向转换器，至少使学术输出可以被工业工具打开和编辑。

---

## 总结：方向一研究差距全景

| Gap | 核心问题 | 影响程度 | 短期可解性 |
|-----|----------|----------|------------|
| Gap 1: DSL 表达力瓶颈 | 品类和特征覆盖不足 | 极高 | 低（需要根本性重设计） |
| Gap 2: 程序合成泛化能力 | 只能填参，不能创造新程序 | 高 | 中（可借鉴 DreamCoder） |
| Gap 3: 缝纫工艺语义缺失 | 只有几何，没有工艺 | 极高 | 低（需要全新建模范式） |
| Gap 4: 跨系统互操作 | 学术与工业格式完全隔离 | 高 | 中（可从单向转换开始） |

**核心洞察**：程序合成路线的四个差距相互关联——DSL 表达力限制了泛化能力（Gap 1→2），DSL 不含工艺信息导致输出不完整（Gap 1→3），不完整的输出又加剧了互操作困难（Gap 3→4）。要系统性地解决这些问题，可能需要**重新思考"版型程序"的定义**——从纯几何描述扩展为包含几何、工艺、约束的多层表示，同时采用可扩展的 DSL 设计（如 library learning 机制），并从一开始就考虑工业格式的兼容性。
