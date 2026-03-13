<!-- markdownlint-disable -->
# Idea 05: 针织指令程序合成——KnitScript-LLM (v1)

## v0→v1 变更说明

本版本基于v0 Rebuttal的七项反馈进行了系统性修订：

1. **放弃全新DSL，改为KnitScript + Python API路线**：回应Rebuttal第4点（Text-to-CadQuery的教训）。KnitScript (Hofmann et al., UIST 2023) 已是一个成熟的、可扩展Python的针织脚本语言，编译到Knitout中间表示。v1不再设计全新KnitCode DSL，而是基于KnitScript构建LLM生成管道，大幅降低生态构建成本。
2. **引入grammar-constrained decoding**：回应Rebuttal第3点。采用SynCode (Ugare et al., 2024) 框架，为KnitScript定义CFG，确保LLM生成的每条指令100%语法正确。针织的"零容错"特性使得constrained decoding从优化项升级为必选项。
3. **明确与McCann编译器系列的差异化定位**：回应Rebuttal第2点。McCann系列 (2016→Lin et al., 2023) 解决的是"3D mesh → machine instructions"的编译问题；本提案解决的是"natural language design description → KnitScript program"的合成问题。两者互补而非竞争：本系统的输出可直接编译到McCann的后端。
4. **缩小scope至2D平面编织图案**：回应Rebuttal第4点。v1聚焦平面编织（flat knitting）的花样合成（stitch patterns），不涉及全3D成型——后者已被McCann系列较好覆盖。
5. **讨论针织仿真的局限性**：回应Rebuttal第5点。承认DiffCloth无法有效模拟针织物，引入替代验证策略（基于Knitout语义的形式化验证 + 物理样品验证）。
6. **数据集策略更新**：回应Rebuttal第6点。利用Neural Inverse Knitting (Kaspar et al., 2019) 的配对数据集方法论，从Ravelry社区的结构化标注中提取训练数据，并用KnitScript渲染器生成合成数据。
7. **评分调整**：根据scope缩小、技术路线明确化重新校准。

---

## 概述

将LLM程序合成范式从梭织服装版型扩展到**针织指令生成**。区别于v0的全新DSL方案，v1采用务实策略：以CMU Textiles Lab的**KnitScript** (Hofmann et al., UIST 2023) 为目标语言——一种已有Python扩展能力、编译到Knitout中间表示的成熟针织脚本语言——训练LLM从自然语言设计描述生成KnitScript程序。结合**grammar-constrained decoding** (SynCode) 确保生成的每条指令语法正确，并通过**Knitout形式语义** (Lin et al., PLDI 2023) 进行可验证性检查。v1聚焦**平面编织花样合成**（stitch pattern generation），不涉及全3D garment shaping——后者已被McCann编译器系列较好覆盖。

## 指向的Gap

- **Gap 1: DSL表达力瓶颈** — GarmentCode只覆盖梭织服装，完全不涉及针织制造范式
- **Gap 13: 非刚性面料的展平失真** — 针织面料的高弹性使传统2D→3D映射方法失效，需要针织专用的设计-制造管道
- **Gap (新增): 自然语言到制造指令的语义鸿沟** — 现有针织工具链（McCann编译器、KnitScript）均要求用户具备专业编程能力，缺少从设计意图到可执行程序的自动转换

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察**：GarmentCode及所有下游方法都假设"裁剪+缝合"的制造模式，而针织市场规模超过$1300亿（T恤、毛衣、运动服、袜子），完全不在其范畴内

2. **计算针织的成熟基础**：McCann et al. (SIGGRAPH 2016) 的针织编译器、Narayanan et al. (SIGGRAPH 2019) 的visual knitting programming、以及Lin et al. (PLDI 2023) 的Knitout形式语义，已构建了一个从高层描述到机器指令的**完整工具链**。但该工具链的输入端仍需专业编程知识。

3. **KnitScript的出现改变了技术路线选择**：Hofmann et al. (UIST 2023) 的KnitScript提供了一种高层针织脚本语言，支持Python扩展、参数化模板、自动化细节处理（如针床管理、纱线载体调度）。其设计理念与GarmentCode高度类似——参数化组件+可组合模板——使得Design2GarmentCode的LLM合成方法论可以更直接地迁移。

4. **Text-to-CadQuery的启示**：与其设计全新DSL（需要从零构建生态），不如复用已有的、有Python扩展能力的语言。KnitScript + Python恰好对应CadQuery + Python的成功模式。

5. **KnitGIST (Hofmann et al., UIST 2020) 的先行验证**：KnitGIST已证明程序合成方法在针织花样生成中的可行性——通过objective-to-tactics映射将功能需求转化为可编织的花样程序。KnitGIST使用了enumeration-based synthesis；v1提出用LLM-guided synthesis替代，利用LLM的自然语言理解能力直接从设计描述生成KnitScript程序。

6. **创新点**：(a) 首次将LLM程序合成引入针织指令生成领域；(b) grammar-constrained decoding保证针织指令的结构正确性（零容错要求）；(c) 基于Knitout形式语义的程序等价性验证

### 与McCann编译器系列的关系（差异化定位）

| 维度 | McCann系列 (2016-2023) | KnitScript-LLM (本提案) |
|------|----------------------|------------------------|
| **输入** | 3D mesh + shaping primitives | 自然语言设计描述 |
| **输出** | 低层机器指令 (Knitout) | 高层KnitScript程序 |
| **核心问题** | 编译（确定性映射） | 合成（意图理解+生成） |
| **用户群** | 有3D建模经验的工程师 | 无编程经验的设计师/编织爱好者 |
| **覆盖范围** | 3D成型（shaping） | 2D花样（stitch patterns） |
| **互补性** | KnitScript-LLM输出可通过KnitScript编译到Knitout，再由McCann后端编译到机器指令 |

两者构成**上下游关系**：本系统负责"设计意图 → KnitScript"，McCann系列负责"Knitout → 机器指令"。

## 详细设计

### 1. 系统架构

```
设计输入（"一个6×6的绞花花样，左交叉，每8行重复一次"）
    ↓
[LLM + Grammar-Constrained Decoding]
    → 输入：自然语言描述 + KnitScript CFG
    → 输出：语法正确的KnitScript程序
    ↓
[Knitout形式语义验证]
    → KnitScript → Knitout → fenced tangle equivalence check
    → 验证程序是否产生预期的拓扑结构
    ↓
[KnitScript可视化渲染]
    → 渲染预览图，用户确认后执行
    ↓
[Knitout编译 → 机器指令]
    → 编译到具体针织机的指令格式
```

### 2. Grammar-Constrained Decoding for KnitScript

KnitScript的语法结构具有以下特性，使其特别适合grammar-constrained decoding：

**a) 有限的操作原语集**：KnitScript的核心操作可表示为有限的CFG产生式规则：

```bnf
<program>    ::= <statement>+
<statement>  ::= <knit_op> | <control_flow> | <function_def>
<knit_op>    ::= "knit" "(" <params> ")"
               | "purl" "(" <params> ")"
               | "tuck" "(" <params> ")"
               | "miss" "(" <params> ")"
               | "xfer" "(" <params> ")"
               | "split" "(" <params> ")"
<control_flow> ::= "for" <var> "in" "range" "(" <expr> ")" ":" <block>
                 | "repeat" "(" <pattern> "," <count> ")"
<params>     ::= <needle_spec> ("," <needle_spec>)*
<needle_spec>::= ("f" | "b") <integer>
```

**b) SynCode集成**：使用SynCode的DFA mask store为KnitScript构建离线查找表。在LLM解码的每一步，SynCode根据已生成的partial program和KnitScript CFG，过滤掉不合法的token，确保输出始终是合法的KnitScript程序。

**c) 针织专用约束层**：在CFG之上叠加针织特定的语义约束（作为post-filter）：
- **针床冲突检查**：同一针位上不能同时执行冲突操作
- **纱线载体连续性**：纱线载体的位置必须连续移动
- **针床容量约束**：每根针上的线圈数不超过物理极限

### 3. 训练数据构建

**策略A：结构化模式提取 (Ravelry → KnitScript)**

Ravelry社区的针织模式虽然多为自然语言，但存在高度结构化的子集：
- 使用标准化缩写（K = knit, P = purl, YO = yarn over, K2tog = knit 2 together）
- 遵循固定格式（"Row 1: K2, P2, repeat to end"）
- 可通过规则解析器转化为KnitScript程序

预期可提取 ~5,000 个 (自然语言描述, KnitScript程序) 配对。

**策略B：合成数据生成 (KnitScript → 渲染 → 反向描述)**

参考Neural Inverse Knitting (Kaspar et al., 2019) 的方法论：
1. 从KnitScript标准库的模板中参数化生成大量花样程序
2. 使用KnitScript可视化渲染器生成预览图
3. 使用VLM为每个渲染结果生成自然语言描述
4. 生成 ~20,000 个合成训练样本

**策略C：现有学术数据集**

- Kaspar et al. (2019) 的配对数据集（约380个花样，含实物图+指令图）
- KnitGIST (Hofmann et al., 2020) 的功能性纹理库
- Knitting Skeletons (Kaspar et al., 2019) 的参数化原语库

### 4. 验证策略（替代物理仿真）

Rebuttal正确指出DiffCloth无法有效模拟针织物。v1采用三层替代验证：

**Level 1: 语法验证 (Grammar Check)**
- SynCode的grammar-constrained decoding保证100%语法正确
- 编译到Knitout验证无编译错误

**Level 2: 形式语义验证 (Formal Semantics)**
- 利用Lin et al. (PLDI 2023) 的Knitout形式语义
- 通过fenced tangle等价性检查验证程序的拓扑正确性
- 验证输出对象是否为合法的编织结构（无悬空线圈、无断裂纱线）

**Level 3: 物理样品验证 (Physical Knitting)**
- 在桌面针织机（如Kniterate）上实际编织生成的程序
- 人工评估花样的正确性和视觉质量
- 仅用于最终评估，不在训练循环中

## 参考文献分析

### 核心参考

1. **KnitScript** (Hofmann et al., UIST 2023)
   - 高层针织脚本语言，支持Python扩展
   - 编译到Knitout中间表示，再编译到具体机器指令
   - 自动处理针床管理、纱线载体调度等底层细节
   - 用户研究证明其可用性（9名机器编织者成功使用）
   - **本提案的目标语言**：LLM生成KnitScript程序，复用其完整的编译后端

2. **KnitGIST** (Hofmann et al., UIST 2020)
   - 程序合成在针织花样生成中的先行验证
   - Objective-to-tactics映射：功能需求（如弹性、透气性）→ 花样策略 → 可编织程序
   - 31次引用，证明了程序合成+针织的学术可行性
   - **v1与其的区别**：KnitGIST使用enumeration-based synthesis，v1使用LLM-guided synthesis

3. **Lin et al.** "Semantics and scheduling for machine knitting compilers" (PLDI 2023)
   - 首次为Knitout定义形式语义（fenced tangle，基于knot theory的扩展）
   - 证明了一组rewrite rules的正确性，可用于编译优化和等价性验证
   - 13次引用，是Knitout生态的理论基础
   - **本提案中的角色**：Level 2验证层——形式化验证生成的针织程序的拓扑正确性

4. **Neural Inverse Knitting** (Kaspar et al., 2019)
   - 从针织物图像反推机器指令的开创性工作
   - 创建了配对数据集（实物图 + 指令图），可作为本提案的训练数据种子
   - 提出了real-synthetic数据混合的理论框架（domain adaptation theory）
   - **本提案中的角色**：数据构建方法论参考 + 种子数据来源

5. **SynCode** (Ugare et al., 2024)
   - Grammar-constrained decoding框架，基于DFA mask store
   - 在JSON/Python/Go上消除100%语法错误
   - 支持任意CFG定义的语言
   - **本提案中的角色**：为KnitScript定义CFG，保证LLM输出的语法正确性

6. **McCann et al.** "A compiler for 3D machine knitting" (SIGGRAPH 2016)
   - 从高层shaping primitives到机器指令的编译器
   - 140次引用，针织编译器的开创性工作
   - **本提案中的角色**：下游编译后端——KnitScript-LLM的输出经Knitout编译到McCann的后端

7. **Narayanan et al.** "Visual knitting machine programming" (SIGGRAPH 2019)
   - 107次引用，首个通用的可视化针织编程界面
   - 引入augmented stitch mesh数据结构
   - **本提案中的角色**：补充参考——展示了针织编程界面设计的多样性

8. **Knitting Skeletons** (Kaspar et al., 2019)
   - 参数化针织原语 + 花样层的组合系统
   - 设计理念与GarmentCode高度类似（参数化组件+可组合模板）
   - **本提案中的角色**：证明参数化组件设计在针织域的可行性；原语库可作为KnitScript模板来源

9. **Zheng et al.** "Automated Knitting Instruction Generation from Fabric Images Using Deep Learning" (2024)
   - 最新的image→knitting instruction工作，扩展了Kaspar 2019的方法
   - 引入五层结构标注和mesh-based推理
   - **本提案中的角色**：证明深度学习方法在针织指令生成领域的持续进展

10. **Design2GarmentCode** (Zhou et al., CVPR 2025)
    - LLM合成DSL程序的方法论可以直接迁移到KnitScript
    - 微调策略、retrieval-augmented generation等技术可复用

### 补充参考

11. **Knitout** — 针织中间表示标准，CMU Textiles Lab维护
12. **Text-to-CadQuery** (2025) — Python API路线的成功案例，启发了v1的技术路线选择
13. **FALCON** (2026) — Grammar-constrained decoding的最新进展
14. **AIDL** (2025) — LLM-native DSL设计原则，可指导KnitScript扩展API的设计
15. **Type-Constrained Code Generation** (Mündler et al., 2025) — 类型系统引导的代码生成，可作为KnitScript类型检查层的参考

## 五维评分

| 维度 | v0分数 | v1分数 | 变更理由 |
|------|--------|--------|---------|
| **新颖性** | 8/10 | 7/10 | KnitGIST (2020) 已在针织领域应用程序合成，Lin et al. (2023) 已建立Knitout形式语义——技术基础比v0评估时更成熟。但"LLM + grammar-constrained decoding → KnitScript"的组合仍是全新的，且自然语言输入端是McCann系列和KnitGIST均未覆盖的 |
| **可行性** | 5/10 | 6/10 | v1采用已有的KnitScript语言（↑↑），有现成的编译后端和可视化工具（↑），SynCode提供了成熟的constrained decoding框架（↑）。但数据获取仍有挑战（Ravelry数据的结构化转换），scope缩小到2D花样降低了工程复杂度（↑） |
| **影响力** | 7/10 | 5/10 | scope从通用针织合成缩小到2D花样合成（↓↓）。与FABRIC-FRONTIER其他Idea的技术协同有限（↓）。但仍开辟了一个学术空白——LLM驱动的针织编程 |
| **清晰度** | 7/10 | 8/10 | 与McCann系列的差异化定位更清晰（↑），技术管道各环节有明确的工具选择（KnitScript / SynCode / Knitout semantics）（↑），验证策略替代了不可行的物理仿真（↑） |
| **证据支撑** | 6/10 | 7/10 | KnitScript用户研究证明了语言的可用性（↑），KnitGIST验证了程序合成在针织域的可行性（↑），Lin et al.的形式语义为验证层提供了理论基础（↑），SynCode在多语言上消除100%语法错误（↑） |
| **总分** | **33/50** | **33/50** | scope缩小导致影响力下降，抵消了可行性和证据方面的提升。总分持平反映了"更可行但更小"的设计权衡 |

## 风险与挑战

1. **数据集构建的工程量**（v0保留，策略更新）：Ravelry的自然语言模式到KnitScript程序的转换仍需大量工程。v1引入三路数据策略（规则解析 + 合成数据 + 学术数据集），但Ravelry的非标准化描述（方言、省略、隐含约定）仍是主要障碍。**缓解策略**：先聚焦最标准化的子集（如使用Craft Yarn Council标准缩写的模式），建立种子数据集后用few-shot learning扩展。

2. **语义约束的覆盖率**（v1新增）：SynCode的CFG保证语法正确，但不保证语义正确。例如，一个语法合法的KnitScript程序可能产生物理上不可编织的结构（如在空针上执行knit操作）。Level 2的Knitout形式语义验证可捕获部分语义错误，但Lin et al.的fenced tangle框架尚未覆盖所有针织约束（如张力、纱线载体路径优化）。**缓解策略**：(a) 在CFG之上添加针织专用的post-filter规则；(b) 通过KnitScript的内置验证器捕获常见错误。

3. **与FABRIC-FRONTIER主线的协同有限**（v0保留，v1承认）：针织与梭织的技术栈几乎完全不同。KnitScript-LLM的方法论（LLM + constrained decoding + formal verification）可迁移，但具体的DSL和工具链无法复用。**缓解策略**：将本Idea定位为方法论验证——如果LLM + grammar-constrained decoding在针织域成功，同样的框架可应用于梭织版型生成中的GarmentCode。

4. **KnitScript的采用度和维护**（v1新增）：KnitScript是一个学术项目（CMU Textiles Lab），其长期维护和社区采用存在不确定性。Knitout作为中间表示的行业采用也仍有限。**缓解策略**：KnitScript编译到Knitout，Knitout可进一步编译到主流针织机格式（Stoll, Shima Seiki）；即使KnitScript停止维护，Knitout生态的延续性由CMU Textiles Lab的持续研究保障（2016-2023持续发表论文）。

5. **scope限制的后果**（v1新增）：聚焦2D花样意味着无法生成完整的针织服装程序（需要3D shaping）。用户可能期望端到端的"描述→完整服装"能力。**缓解策略**：明确定位为花样设计工具，3D shaping交由McCann编译器系列处理。长期路线图中可探索KnitScript-LLM + McCann编译器的端到端集成。
