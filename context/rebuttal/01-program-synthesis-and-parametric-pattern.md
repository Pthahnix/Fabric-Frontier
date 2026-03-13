# Rebuttal: Direction 1 — Program Synthesis与参数化版型生成

> 针对Gap 1-4的系统性反驳。核心论点：四个Gap过度聚焦于GarmentCode DSL的内部局限，忽略了program synthesis领域近两年的突破性进展，以及DSL范式之外的替代路径。

---

## Gap 1: DSL表达力瓶颈

**原始论断**：GarmentCode等参数化DSL存在固有的表达力天花板，无法覆盖真实世界服装版型的全部复杂性，需要专家手动扩展原语集合。

### 反驳1："天花板效应"论断过于悲观——Library Learning已证明DSL可自动扩展

Gap分析将DSL表达力视为一个静态问题，仿佛DSL的原语集合一旦设计完成就永远固定。这忽视了program synthesis领域最重要的近期进展：**自动库学习（library learning）**。

LILO (Grand et al., ICLR 2024) 在多个domain上证明了DSL可以通过自动抽象发现持续扩展。在REGEX domain，LILO达到93.2%的解决率，而DreamCoder仅为45.6%——提升幅度超过一倍。更关键的是其机制：LILO的Stitch compression比DreamCoder快**1000-10000倍**，使得library learning从学术演示变为工程可行的技术。LILO的offline synthesis实验直接证明了自动发现的库函数的价值：使用LILO库的synthesis性能比基础DSL**高出42.27分**。

这意味着GarmentCode的表达力瓶颈不是DSL范式的固有缺陷，而是**缺少自动扩展机制**。如果将LILO的library learning框架应用于GarmentCode——让系统从大量版型实例中自动发现可复用的高层抽象（如"公主线结构"、"插肩袖连接"、"育克分割"）——表达力瓶颈就变成了一个工程问题而非基础性限制。

此外，LILO的AutoDoc机制揭示了一个微妙但重要的发现：**有命名的抽象比匿名抽象效果好得多**（+9.73分 vs -30.60分）。这暗示了一条实践路径：自动发现的版型原语必须带有人类可理解的语义标签（"dart"、"pleat"、"gusset"），否则反而会损害LLM的使用能力。

### 反驳2：问题可能不是"DSL表达力不够"，而是"DSL不是为LLM设计的"

AIDL (Mistry et al., 2025) 提出了一个根本性的paradigm shift：**不是调优模型以适应现有图形系统，而是为语言模型量身构建图形系统**（原文："What if, instead of tuning a model for a graphics system, we build a graphics system tailored for language models?"）。

AIDL的实验结果支持了这一论点：在图形生成任务上，AIDL的CLIP score达到28.90，优于OpenSCAD的27.32。更具说服力的是负面案例——当LLM尝试使用工业CAD系统FeatureScript时，"LLM failed to generate syntactically correct programs in **almost all cases**"。这不是LLM能力不足，而是DSL设计与LLM认知模式的不匹配。

将此洞察应用于服装领域：GarmentCode的问题可能不是"表达力不够"，而是其DSL设计没有考虑LLM的生成特性。GarmentCode使用的是面向人类专家的声明式规范——参数命名、嵌套结构、隐式约束——这些对LLM来说都是额外的认知负担。按AIDL的思路，应该设计一种**LLM-native的服装描述语言**，利用LLM擅长的序列化生成、模式匹配和自然语言理解能力。

### 反驳3：遗漏了非DSL替代方案——GarmageNet等图像化表征

Gap分析过度聚焦于program synthesis范式内部的问题，完全忽视了**范式外的替代方案**。GarmageNet等方法将版型表示为multi-channel image（每条线段一个channel），完全绕过了DSL的表达力问题。这类图像化表征方案的存在说明DSL本身可能不是唯一正确的路线——如果表达力瓶颈确实不可逾越（我们论证了它是可以逾越的），图像化方案提供了一条完全不同的escape path。

Gap分析应该评估：DSL表达力瓶颈是需要在范式内部解决，还是需要切换范式？两条路径的trade-off是什么？目前的分析缺少这一层次的思考。

### 反驳4：Python原生DSL天然继承LLM的code generation prior

Text-to-CadQuery (Kim et al., 2025) 提供了一个极具说服力的数据点：基于Python的DSL（CadQuery），即使使用**仅124M参数的模型**，也能在CAD生成任务上超越使用专用Transformer架构的Text2CAD（363M参数）。论文的解释直截了当："Since LLMs already excel at Python generation... fine-tuning them on Text-to-CadQuery data proves highly effective."

这暗示了一条极具实践价值的路径：将GarmentCode重构为**Python原生API**（可称为"GarmentQuery"），而非独立DSL。这样做的好处是三重的：(i) 继承LLM在Python代码生成上的海量先验知识，(ii) 获得Python生态系统的所有工具支持（调试、测试、包管理），(iii) 降低新用户的学习曲线。DSL的"表达力瓶颈"可能只是"Python表达力瓶颈"的一个不必要的严格化。

---

## Gap 2: 泛化能力——从模板填参到真正的程序合成

**原始论断**：当前方法本质上是"模板填参"，无法实现真正的程序合成，即从零生成结构性新颖的版型。

### 反驳1："模板填参vs真正合成"是一个假二分法

Gap分析将程序合成的能力谱简化为两个极端：要么是trivial的参数替换，要么是从空白状态完全从头生成。这忽视了**中间地带**的丰富可能性，而近期研究恰恰证明中间地带是最有成效的。

AIDL展示了一种分层架构：LLM负责高层语义结构（物体的组成和关系），约束求解器处理底层几何精度（尺寸、位置、对齐）。LLM不需要"从零合成"所有细节——它只需要正确指定结构，然后将精确计算委托给专门的solver。这种"LLM做创意决策，solver做精确计算"的分工模式在服装版型领域有天然的对应：LLM决定版型的结构（几片、什么形状的连接关系），solver计算具体的曲线控制点和缝份。

FALCON (Li et al., 2026) 则从另一个角度证明了可行性：通过grammar-constrained decoding + repair layers + adaptive sampling，GPT-4o在TSP问题上的可行性从39%提升到**100%**。关键洞察是"将挑战分解为syntactic validity和semantic feasibility"（原文："Separate the challenge into syntactic validity and semantic feasibility"）。应用于版型生成：语法正确性（合法的GarmentCode程序）和语义可行性（物理上可缝制的版型）可以分别保证，不需要LLM同时处理两者。

### 反驳2：LLM对符号程序具有真正的语义理解能力

SGP-Bench的实验提供了强有力的证据：LLM在坐标扰动下展现出**>80%的一致性**，证明其对符号程序具有genuine semantic understanding，而非简单的模式匹配或记忆。更具说服力的是SIT (Symbolic Instruction Tuning) 的跨域迁移效果：在符号程序理解上的训练不仅提升了目标任务，还提升了通用推理能力（AGIEval +6.6, GSM8k +2.5）。

这意味着LLM处理版型程序的能力可能比Gap分析假设的要强得多。LLM不仅能"填参数"，还能理解参数修改对最终几何形状的语义影响——这正是"真正的程序合成"所需要的核心能力。

### 反驳3：对DreamCoder的引用已显著过时

Gap分析引用DreamCoder (Ellis et al., 2021) 作为program synthesis泛化能力有限的证据。但DreamCoder已有五年历史，其继承者LILO (2024) 在效率和效果上都有数量级的提升。用2021年的结果来评估2026年的program synthesis能力，相当于用GPT-2的表现来评估当前LLM的能力——结论方向可能正确，但程度判断严重失真。

### 反驳4：Tool-augmented方案可在不重新训练的情况下扩展泛化能力

CAD-Assistant (Sarda et al., 2024) 展示了一种training-free的tool augmentation方案，其关键能力是"can operate beyond the simple set of CAD commands captured by existing datasets"（原文）。这意味着泛化能力不一定来自模型内部——通过提供正确的工具集（几何计算库、约束检查器、版型验证器），LLM可以在推理时扩展其操作空间。

CAD-Assistant中，constraint checker单独就将性能从PF1 0.747提升到0.979——说明**外部验证工具比模型内部能力更重要**。对于服装版型，这意味着一个好的缝制可行性检查器可能比一个更强的生成模型更有价值。

---

## Gap 3: 缝纫工艺语义的缺失

**原始论断**：当前系统缺乏对缝纫工艺约束的建模能力——缝份宽度、对位标记（notch）、grain line方向、工序依赖等制造知识无法被程序合成方法捕获。

### 反驳1：FALCON的三层可行性保证架构直接适用于缝纫约束

FALCON提出的三层架构为缝纫工艺约束提供了一个现成的解决框架：

- **Layer 1: Grammar-constrained decoding** — 确保生成的版型程序在语法上合法。对应服装领域：所有版型片段必须是闭合多边形，缝合边必须成对出现，grain line必须存在。
- **Layer 2: Feasibility repair** — 在生成后修复语义违规。对应服装领域：缝合边长度不匹配时自动调整（ease分配）、缝份宽度不符合标准时自动修正、notch位置与缝合边不对应时重新计算。
- **Layer 3: Adaptive sampling** — 根据约束满足度调整生成策略。对应服装领域：如果某类结构（如设计省道转移）频繁违反约束，增加该区域的采样温度或切换策略。

FALCON在combinatorial optimization问题上达到**100%可行性**，而GPT-4o裸模型仅39%。BOPO训练后的模型在修复前就达到81-97%的可行性。这些数字表明，缝纫约束的满足不需要LLM"理解"工艺——只需要正确的约束检查和修复机制。

### 反驳2：AIDL的约束系统有直接的缝纫语义类比

AIDL实现了一套几何约束系统，包括Coincident（重合）、Equal（等长）、Symmetric（对称）、Parallel（平行）等。这些约束在缝纫工艺中有直接的语义对应：

| AIDL约束类型 | 缝纫工艺对应 |
|---|---|
| Coincident | 缝合点对齐（notch matching） |
| Equal | 缝合边等长（seam allowance matching） |
| Symmetric | 对称省道（symmetric darts） |
| Parallel | Grain line与布边平行 |
| Perpendicular | 缝份与缝合线垂直 |

关键洞察：缝纫语义可以**编码为约束**，由constraint solver而非LLM处理。LLM只需要指定"这两条边需要缝合"，solver负责计算具体的缝份宽度、ease分配和notch位置。这种分工既利用了LLM的语义理解能力，又避免了让LLM处理精确数值计算的弱点。

### 反驳3：工具增强方案——不需要把所有工艺知识编码进DSL

CAD-Assistant的核心洞察是：LLM不需要内化所有领域知识，只需要知道何时调用什么工具。将此思路应用于缝纫工艺：

- **缝份计算器**（输入：面料类型+缝合方式 → 输出：推荐缝份宽度）
- **Notch生成器**（输入：两条缝合边 → 输出：对位标记位置和类型）
- **Grain line检查器**（输入：版型片段+面料信息 → 输出：最优grain line方向和偏差警告）
- **工序排序器**（输入：所有缝合操作 → 输出：最优工序DAG）

这些工具可以封装成熟的工艺知识，LLM通过function calling调用它们。CAD-Assistant中，constraint checker单独就带来了PF1从0.747到0.979的提升——**单个外部工具的效果超过了模型架构的改进**。

### 反驳4："DAG结构化知识"并非未解决的问题

Gap分析正确地指出制造工序具有有向无环图（DAG）结构，但将其表述为一个开放性挑战。实际上，**工序排序和约束满足在工业工程领域已有数十年的成熟方法**——关键路径法（CPM）、PERT图、约束满足问题（CSP）求解器都是标准工具。问题不在于"能否对工序DAG建模"，而在于"如何获取足够的训练数据来学习工序约束"。这是一个数据工程问题，不是算法问题。

---

## Gap 4: 跨CAD互操作性

**原始论断**：程序合成方法的输出无法与工业CAD系统（CLO3D, Gerber, Lectra）互操作，缺乏标准化的中间表示。

### 反驳1：Textile IR不应被轻易否定

Gap分析将Textile IR描述为"仅仅是position paper"，但这种评估缺乏严谨性。实际上，该论文包含了相当具体的技术内容：

- **正式的节点类型规范**：PatternPiece, SeamEdge, Dart, Notch, MaterialRegion——每种类型都有明确的属性定义
- **双向流转示例**：展示了从设计到制造再返回的完整数据流
- **复合不确定性分析**：讨论了几何精度、材料属性和工艺参数的不确定性传播
- **可证伪性标准**：提出了验证IR完备性和正确性的具体方法

这远不止是一个概念提案。将其与其他领域的IR发展历程对比（如LLVM IR从论文到工业标准历时约5年），Textile IR处于一个合理的早期阶段。问题不是"IR不存在"，而是"IR尚未获得足够的社区采用"——这是一个生态系统问题，不是技术问题。

### 反驳2：LLM可以直接使用现有CAD API，无需中间格式

Text-to-CadQuery和CAD-Assistant都证明了LLM可以直接调用现有的CAD Python API（FreeCAD, CadQuery, OpenCascade），无需中间格式转换。CAD-LLaMA的SPCC（Structured Parametric CAD Code）达到了**99.90%的语法成功率**。

将此应用于服装领域：GarmentCode的输出不一定需要通过专门的翻译器转换为DXF——可以让LLM直接生成DXF兼容的Python代码。DXF格式的核心实体（LINE, ARC, POLYLINE, TEXT）在Python中有成熟的库支持（ezdxf）。互操作性问题可能被人为复杂化了。

### 反驳3：SVG/DXF生成已基本解决，工程量被高估

SVG生成在近两年已取得显著进展：SVGen使用3B fine-tuned模型在FID上超越GPT-4o（27.82 vs 47.73），StarVector实现了高质量的矢量图形生成。DXF在结构上与SVG高度相似——都是2D矢量图形格式，都使用基本几何元素（线段、圆弧、样条曲线）的组合。

从SVG生成到DXF生成的技术距离远小于Gap分析所暗示的。主要差异在于：(i) DXF需要图层管理（但这是元数据，不是几何问题），(ii) DXF需要精确的单位系统（但这是工程规范，不是生成难度），(iii) DXF需要特定的实体类型（但POLYLINE和SPLINE已覆盖90%以上的版型几何）。

---

## 总结与重新评估建议

Direction 1的四个Gap共享一个系统性偏差：**将GarmentCode的当前局限等同于program synthesis范式的固有局限**。近两年的研究进展提供了至少三条明确的外部解决路径：

1. **LILO式library learning**：自动扩展DSL表达力，速度比前代方法快1000-10000倍，效果提升42.27分
2. **AIDL式LLM-native DSL设计**：不是让模型适应DSL，而是为LLM重新设计DSL，从根源上解决可用性问题
3. **CAD-Assistant式tool augmentation**：绕过DSL限制，通过外部工具（约束检查器、几何计算器、格式转换器）扩展能力，单个工具就能将性能从0.747提升到0.979

建议将四个Gap的严重性等级下调，或至少将其从"基础性限制"重新分类为"工程挑战"。同时建议增加一个新的Gap：**"如何将这些已证明有效的通用技术适配到服装版型的特定领域"**——这才是真正需要解决的问题。
