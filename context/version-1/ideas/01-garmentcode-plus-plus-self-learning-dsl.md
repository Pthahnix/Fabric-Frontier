<!-- markdownlint-disable -->
# Idea 01: GarmentCode++ 自学习DSL扩展 (v1)

## v0→v1 变更说明

本版本基于v0 Rebuttal的六项反馈进行了系统性修订：

1. **技术基线更新**：从DreamCoder (NeurIPS 2021) 迁移至LILO (ICLR 2024)。LILO在速度上提升1000-10000x，库质量提升+42.27（相对base DSL），并引入了对本提案至关重要的AutoDoc机制。
2. **AutoDoc升级为核心模块**：v0中缺失语义命名机制。LILO实验证明，匿名抽象不仅无用，反而导致性能下降-30.60（REGEX域）。GarmentCode++必须为每个学到的版型原语生成人类可读的语义名称（如"princess_seam_split"而非"fn_42"），否则LLM合成器无法正确部署学到的抽象。
3. **物理仿真验证层**：引入DiffCloth/DiffAvatar作为external verifier，构建"生成→仿真→反馈"闭环，回应GVU理论关于Hallucination Barrier的警告。
4. **AIDL替代路径讨论**：新增与AIDL (2025) LLM-native DSL设计路径的对比分析，论证library learning与DSL重设计的互补性而非对立性。
5. **可自动化质量指标**：定义基于物理仿真的具体评估指标，回应"服装版型的测试是什么"这一关键问题。
6. **评分调整**：根据新证据和Rebuttal建议重新校准五维评分。

---

## 概述

受LILO（Grand et al., ICLR 2024）neurosymbolic library learning框架的启发，提出GarmentCode++：一个能够**自动发现、语义命名和物理验证**版型构造子程序的自学习DSL系统。与当前GarmentCode需要专家手工添加新模板不同，GarmentCode++通过三阶段循环——**LLM引导合成 → Stitch符号压缩 → AutoDoc语义文档化**——在解决新设计任务的过程中自动归纳出可复用的版型构造原语。关键创新在于引入**可微物理仿真**（DiffCloth/DiffAvatar）作为外部验证器，确保学到的抽象不仅在代码层面合法，更在物理层面可穿着，从而突破GVU理论所预警的"自生成+自评估"停滞陷阱。

## 指向的Gap

- **Gap 1: DSL表达力瓶颈** — GarmentCode仅覆盖有限品类（上衣/裤装/裙装/连衣裙），无法表示口袋、特殊领型、非流形结构等。扩展方式为手工添加模板，不可扩展。
- **Gap 2: 程序合成的泛化能力** — 当前方法（Design2GarmentCode, ChatGarment）本质是"模板填参"而非真正的程序合成，无法创造新的版型拓扑结构。
- **Gap 3: 学到的抽象缺乏语义可读性** — 即使实现了library learning，匿名的lambda抽象（如DreamCoder的输出）对LLM合成器和人类设计师均不可用，LILO的实验数据（-30.60性能下降）证实了这一问题的严重性。
- **Gap 4: 缺乏外部验证信号** — Self-Improving Coding Agent (17%→53% SWE-Bench) 和 Darwin Godel Machine (20%→50%) 的成功均依赖明确的外部验证信号（测试通过/不通过）。版型领域缺乏等价的自动化验证机制。

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察**：GarmentCode的表达力由其预定义的组件库决定。ChatGarment为了增加覆盖范围创建了GarmentCodeRC扩展版，但扩展方式是"逐个品类手工添加"。Design2GarmentCode明确承认"cannot substantially alter GarmentCode's underlying structure"。

2. **技术基线演进**：在通用程序合成领域，library learning已从DreamCoder演进到LILO。LILO的关键突破包括：
   - **双系统搜索**：LLM引导搜索（快速，利用预训练知识）+ 枚举搜索（慢速，发现域特定表达），两者互补
   - **Stitch压缩**：比DreamCoder的压缩算法快1000-10000x，内存效率高100x，使得每轮迭代可从头重新推导整个库（"deep refactoring"）
   - **AutoDoc**：为学到的抽象自动生成人类可读名称和文档字符串。实验证明AutoDoc对REGEX域的性能提升为+9.73，且消除了匿名抽象导致的-30.60性能下降

3. **迁移**：将LILO的neurosymbolic library learning框架迁移到版型DSL领域。版型构造中存在大量可复用的子模式（如"一字领构造"、"公主线分割"、"活褶构造"），这些模式在不同款式中反复出现，非常适合被Stitch自动识别并由AutoDoc命名。

4. **外部验证器的必要性**：GVU理论（2025）指出，当同一系统既生成又评判输出时，self-improvement会停滞。对版型领域，外部验证器应为可微物理仿真（DiffCloth, Li et al., 2022; DiffAvatar, Li et al., CVPR 2024）。DiffCloth基于Projective Dynamics，支持干摩擦接触的梯度计算，已在逆向设计（inverse design）任务上展示了有效性。DiffAvatar进一步展示了直接在2D pattern空间中优化版型的能力，与GarmentCode++的pattern-level library learning天然互补。

5. **创新点**：(a) 首次将LILO框架应用于几何约束密集的版型DSL领域；(b) AutoDoc为版型原语提供制版术语级别的语义命名；(c) 可微仿真作为外部验证器构建闭环，突破hallucination barrier。

### 相关工作对比

| 方法 | 扩展方式 | 语义命名 | 外部验证 | 局限 |
|------|---------|---------|---------|------|
| GarmentCode (SIGGRAPH 2023) | 专家手工定义 | 手工命名 | 无 | 新品类需要大量编程 |
| ChatGarment GarmentCodeRC | 专家手工扩展 | 手工命名 | 无 | 仍需"expanding garment type coverage" |
| Design2GarmentCode | 不扩展DSL | — | 无 | 限于现有模板空间 |
| DreamCoder (NeurIPS 2021) | 自动library learning | 匿名lambda | 无 | 仅用于简单域，无语义命名 |
| LILO (ICLR 2024) | 自动library learning | AutoDoc | 无 | 仅在REGEX/CLEVR/LOGO上验证 |
| AIDL (2025) | LLM-native DSL设计 | 语义化操作符 | Constraint solver | 不学习新抽象，静态DSL |
| **GarmentCode++ (本提案)** | 自动library learning | AutoDoc + 制版术语 | 可微物理仿真 | 需要版型域的适配 |

### 与AIDL路径的对比分析

AIDL（2025）提出了一条根本不同的路径：与其让现有DSL自学习扩展，不如**从头为LLM设计一种新的DSL**。AIDL为2D CAD设计了solver-aided hierarchical DSL，实现了四个设计目标——dependencies、constraints、semantics、hierarchy——并配合constraint solver将空间推理从LLM中卸载。实验表明，即使LLM从未在训练数据中见过AIDL代码，其生成质量也优于LLM熟悉的OpenSCAD（CLIP score 28.90 vs 27.32）。

**我们的立场**：Library learning与LLM-native DSL设计并非对立，而是互补的两个维度：

- **AIDL回答的问题**：DSL的语法和语义结构应如何设计，才能让LLM更有效地生成代码？
- **GarmentCode++回答的问题**：DSL的原语库应如何自动增长，才能覆盖不断涌现的新设计需求？

具体而言，GarmentCode++可以将AIDL的设计原则融入其base DSL的改造中——例如采用AIDL的hierarchical structure组织版型组件、使用constraint solver处理几何约束、通过synonymous operators降低LLM的语法错误率——同时在此基础上叠加library learning来自动扩展原语库。两者的结合将产生一种**既对LLM友好、又能自我进化**的版型DSL。

在实验设计中，我们计划设置ablation study对比以下配置：
1. GarmentCode (baseline) — 现有静态DSL
2. GarmentCode + AIDL重设计 — 静态但LLM-native的DSL
3. GarmentCode + Library Learning — 动态但非LLM-native的DSL
4. GarmentCode++ (full) — AIDL设计原则 + Library Learning + AutoDoc + 物理验证

## 详细设计

### 1. 系统架构

```
输入：新设计任务（图像/文本 → 目标版型）
    ↓
[双系统合成器] — LLM引导搜索 + 枚举搜索，使用当前DSL库
    ↓ 成功 → 候选程序集合
    ↓ 失败 → 标记"表达力不足"，进入下一轮Wake-Sleep
    ↓
[Stitch符号压缩] — 在候选程序集合中识别最优lambda抽象
    ↓
[AutoDoc语义文档化] — 为每个新抽象生成制版术语级别的名称和文档
    ↓                    （如 "princess_seam_split(panel, curve, ease)"）
[物理仿真验证器] — DiffCloth/DiffAvatar仿真，计算质量指标
    ↓ 通过 → 添加到DSL库
    ↓ 不通过 → 反馈梯度信息，优化程序参数后重新验证
    ↓
[库更新] — 使用扩展后的DSL重新压缩已有程序（deep refactoring）
```

### 2. 核心组件

**a) 双系统合成器（Dual-System Synthesizer）**

继承LILO的双系统架构：
- **LLM引导搜索（System 1）**：使用预训练LLM（如GPT-4/Claude）作为快速近似搜索模型。输入为设计描述 + 当前DSL库的AutoDoc文档 + 已解决任务的程序示例。LLM利用预训练中积累的服装设计知识（如"princess seam"、"dart"等术语的语义理解）来引导搜索。
- **枚举搜索（System 2）**：使用recognition network引导的PCFG枚举搜索，在程序空间中发现结构上不同于LLM输出的新程序。这对发现版型领域的特殊结构尤为重要——例如某些拓扑连接方式可能在自然语言中难以描述但在程序空间中可以枚举发现。

**b) Stitch符号压缩（Symbolic Compression）**

直接使用LILO验证过的Stitch压缩系统：
- 在成功合成的程序集合中，通过branch-and-bound搜索识别最优lambda抽象
- 比DreamCoder压缩算法快1000-10000x，通常在单CPU上数秒内完成
- 支持"deep refactoring"——每轮迭代从base DSL重新推导整个库，丢弃早期发现的次优抽象

**c) AutoDoc语义文档化（Auto-Documentation）**

LILO的AutoDoc机制对GarmentCode++至关重要，但需要领域适配：

- **制版术语对齐**：AutoDoc的LLM prompt中加入制版术语表（patternmaking glossary），引导命名结果使用标准制版术语。例如，当Stitch发现一个在多个上衣程序中出现的将前片沿胸点到肩线的曲线分割成两片的子模式时，AutoDoc应将其命名为`princess_seam_split`而非`panel_curve_division`。
- **层次化命名**：借鉴LILO在REGEX域中的经验（vowel → consonant → replace_consonant_with_substring），版型抽象也应支持层次化语义级联。例如：`dart(position, width, depth)` → `waist_dart(panel, ease)` → `fitted_bodice(measurements, dart_config)`。
- **命名质量验证**：参考LILO在LOGO域中观察到的AutoDoc语义错误（如将polygon命名为"looping move and rotate"），引入self-consistency检查——让LLM根据AutoDoc生成的名称预测子程序的行为，与实际行为对比，不一致时重新命名。

**d) 物理仿真验证器（Physics Simulation Verifier）**

这是v1新增的核心模块，回应GVU理论的Hallucination Barrier警告：

- **验证流程**：
  1. 新学到的子程序生成2D pattern panels
  2. DiffCloth/DiffAvatar将panels缝合并在SMPL body上仿真
  3. 计算可自动化质量指标（见下文）
  4. 通过/不通过决定是否将子程序添加到库中

- **可自动化质量指标**：
  | 指标 | 描述 | 阈值来源 |
  |------|------|---------|
  | 穿透检测通过率 | 仿真后cloth mesh与body mesh的碰撞检测 | 工业标准：零穿透 |
  | 缝合边匹配度 | 对应缝合边的长度差异 ≤ 容差 | GarmentCode现有约束 |
  | 应力分布 | Von Mises应力的空间分布是否在面料承受范围内 | 面料物理参数 |
  | 悬垂性评分 | 仿真后garment的drape coefficient与目标面料的偏差 | 面料数据库 |
  | 几何合法性 | Panel闭合性、无自交、可展性（developability） | 数学约束 |

- **梯度反馈**：DiffCloth/DiffAvatar提供可微梯度，当质量指标不达标时，可将梯度回传至程序参数进行优化。例如，如果某个新发现的`flared_skirt_panel`子程序在仿真后产生过度拉伸，梯度信息可指导调整其radius和flare_angle参数的默认范围。

- **计算效率考量**：DiffCloth基于Projective Dynamics，前向仿真已高度优化；梯度计算使用迭代求解器，比标准线性求解器有显著加速（论文报告85x speedup in gradient computation）。DiffAvatar基于DiffXPBD，同样具备高效的梯度计算能力。单个版型的仿真验证预计在秒级完成，不会成为library learning循环的瓶颈。

### 3. 训练流程（LILO-Style循环）

**Phase 1: 双系统合成（Search）**
- 给定设计任务集合（从时尚电商、设计师手稿、版型教材收集的超出当前DSL表达力的设计）
- LLM引导搜索：使用当前DSL的AutoDoc文档构建few-shot prompt，采样多个候选程序
- 枚举搜索：使用recognition network引导的PCFG在程序空间中搜索
- 收集成功合成的程序集合Π

**Phase 2: Stitch压缩（Compression）**
- 对Π执行Stitch压缩，识别可复用的lambda抽象
- 支持从base DSL重新推导整个库（deep refactoring），避免次优抽象累积

**Phase 3: AutoDoc文档化（Documentation）**
- 对每个新发现的抽象，使用制版术语表增强的prompt生成语义名称和文档
- 执行self-consistency检查，确保命名准确反映子程序行为
- 命名结果支持层次化级联

**Phase 4: 物理仿真验证（Verification）**
- 对每个新抽象生成的panel进行DiffCloth/DiffAvatar仿真
- 计算五项质量指标
- 通过验证的抽象添加到DSL库；不通过的记录失败原因，供后续分析

**迭代**：重复Phase 1-4，DSL库逐步增长。每轮迭代后评估DSL表达力覆盖率的增长。

### 4. 数据需求

- **初始DSL**：现有GarmentCode及其组件库，按AIDL设计原则改造（hierarchical structure、constraint-based参数化、synonymous operators）
- **任务集**：需要大量**超出当前DSL表达力**的设计任务。来源包括：
  - 时尚电商图片（标注为"口袋型"、"特殊领型"等品类标签）
  - 版型教材中的构造图解（转化为程序合成任务规格）
  - 设计师手稿和工业制版数据
- **制版术语表**：从ISO 4915/4916（缝纫针迹和缝型标准）、版型教材术语表等来源构建
- **面料物理参数数据库**：用于仿真验证器的材料属性设定

### 5. Base DSL的AIDL化改造

在启动library learning之前，对GarmentCode的base DSL进行AIDL启发的改造：

1. **Hierarchical Structure**：版型组件按garment → section → panel → feature的层次组织，支持LLM进行模块化推理
2. **Constraint-Based参数化**：用声明式约束（如`Coincident(sleeve_cap.top, bodice.shoulder_point)`）替代命令式坐标计算，将几何推理从LLM中卸载
3. **Semantic References**：所有几何实体使用语义化名称引用（如`front_panel.bust_point`而非坐标索引）
4. **Synonymous Operators**：允许同义操作符（如`Dart`和`Tuck`在特定上下文中等价），降低LLM语法错误率

这一改造确保base DSL本身就是LLM-friendly的，使后续的library learning在更优的基础上进行。

## 参考文献分析

### 核心参考

1. **LILO** (Grand et al., ICLR 2024)
   - neurosymbolic library learning框架：LLM引导搜索 + Stitch压缩 + AutoDoc
   - 在REGEX/CLEVR/LOGO三个域上验证，相比DreamCoder在任务解决率上提升+33.14/+2.26/+20.42
   - AutoDoc消除匿名抽象的-30.60性能下降，证明语义命名的必要性
   - Stitch压缩1000-10000x加速，使deep refactoring成为可能
   - **直接可迁移**：框架的三模块架构（搜索-压缩-文档化）通用性强

2. **DiffCloth** (Li et al., ACM TOG 2022)
   - 基于Projective Dynamics的可微布料仿真，支持干摩擦接触的梯度计算
   - 应用：system identification、trajectory optimization、inverse design、real-to-sim transfer
   - 梯度计算的迭代求解器带来显著加速
   - **本提案中的角色**：外部物理验证器，提供自动化的pass/fail信号和梯度反馈

3. **DiffAvatar** (Li et al., CVPR 2024)
   - 首次在可微仿真中实现直接在2D pattern空间中优化版型
   - 使用control cage representation正则化2D pattern空间
   - 证明了"仿真梯度 → pattern参数优化"路径的可行性
   - **本提案中的角色**：为library learning提供pattern-level的梯度反馈

4. **AIDL** (2025)
   - LLM-native DSL设计：solver-aided、hierarchical、semantic
   - 核心洞察：DSL的语法结构设计对LLM性能有决定性影响
   - 即使LLM未经AIDL训练，AIDL生成质量也优于训练数据中包含的OpenSCAD
   - **本提案中的角色**：base DSL改造的设计原则来源

5. **GarmentCode** (Korosteleva & Sorkine-Hornung, SIGGRAPH 2023)
   - 提供初始DSL基础，其参数化设计（组件+接口）天然适合子程序抽象
   - DSL使用Python实现，组件以类的形式组织

6. **Design2GarmentCode** (Zhou et al., CVPR 2025) / **ChatGarment** (Bian et al., CVPR 2025)
   - 展示了LLM合成GarmentCode程序的可行性
   - 明确承认DSL表达力瓶颈——正是本提案要解决的核心问题

### 补充参考

7. **Stitch** (Bowers et al., 2023) — LILO使用的符号压缩系统
8. **GVU Theory** (2025) — Hallucination Barrier理论，论证external verifier的必要性
9. **Self-Improving Coding Agent** (2025) — 自改进范式中外部验证信号的关键作用
10. **CAD-LLaMA** (2025) — hierarchical description使准确率从39.17%跃升至80.41%
11. **REGAL** (Stengel-Eskin et al., 2024) — LLM重构程序以发现可泛化抽象的另一种方法
12. **Leroy** (Bellur et al., 2024) — 将library learning扩展到命令式编程语言

## 五维评分

| 维度 | v0分数 | v1分数 | 变更理由 |
|------|--------|--------|---------|
| **新颖性** | 9/10 | 8/10 | LILO已将library learning推进到neurosymbolic阶段，本Idea的核心贡献从"首次引入library learning"修正为"首次将LILO框架适配到几何约束密集的版型DSL + 物理仿真验证闭环"。仍然是全新的研究方向，但技术基线的成熟度降低了纯概念层面的新颖性 |
| **可行性** | 5/10 | 6/10 | LILO的工程成熟度远高于DreamCoder（↑）：Stitch压缩秒级完成、AutoDoc有现成实现、代码开源。DiffCloth/DiffAvatar已在inverse design任务上验证了梯度反馈的有效性（↑）。但服装版型的域复杂度仍远超LILO验证过的REGEX/CLEVR/LOGO（↓），整体略有提升 |
| **影响力** | 9/10 | 9/10 | 维持——若成功，将根本解决DSL表达力瓶颈，使版型生成的设计空间从有限到理论上可自动扩展。物理仿真验证层进一步提升了学到的抽象的实用价值 |
| **清晰度** | 7/10 | 8/10 | v1补充了LILO/AIDL的详细讨论、物理验证层的具体指标定义、与AIDL路径的对比分析、ablation study设计。方案完整性显著提升 |
| **证据支撑** | 6/10 | 7/10 | 核心引用从过时的DreamCoder更新为LILO（↑）。DiffCloth/DiffAvatar在inverse design上的成功为物理验证层提供了直接证据（↑）。AIDL的实验结果为DSL设计原则提供了量化依据（↑）。但仍缺乏版型域的直接实验证据（→） |
| **总分** | **36/50** | **38/50** | |

## 风险与挑战

1. **域复杂度鸿沟**（v0保留，细化）：LILO在REGEX/CLEVR/LOGO上的成功是否能迁移到版型构造，仍是核心不确定性。版型的几何闭合、缝合拓扑、物理可行性约束使搜索空间远大于LILO验证过的任何域。**缓解策略**：(a) 通过AIDL化改造简化base DSL，降低搜索空间复杂度；(b) 利用LLM的制版领域知识引导搜索方向；(c) 分阶段增加复杂度（先简单品类如A-line skirt，后复杂品类如tailored jacket）。

2. **AutoDoc在版型域的准确性**（v1新增）：LILO在LOGO域中已观察到AutoDoc的语义错误（如将polygon命名为"looping move and rotate"）。版型域的专业性更强，AutoDoc可能生成不准确的制版术语。**缓解策略**：(a) prompt中注入标准制版术语表；(b) self-consistency验证；(c) 可选的人类专家审核环节。

3. **物理仿真验证的计算成本**（v1新增）：虽然DiffCloth/DiffAvatar在单个版型上的仿真效率较高，但library learning循环中需要验证大量候选抽象。**缓解策略**：(a) 先执行低成本的几何验证（闭合性、无自交），仅对通过几何验证的子程序执行物理仿真；(b) 使用粗网格和较少仿真步数进行初筛，精细仿真仅用于最终入库验证。

4. **任务集构建**（v0保留）：需要大量"刚好超出当前DSL"的设计任务。**缓解策略**：(a) 从版型教材的递进练习中提取任务序列；(b) 使用VLM从时尚电商图片自动提取版型特征描述作为任务规格。

5. **评估困难**（v0保留，细化）：DSL"表达力增长"的衡量指标。**缓解策略**：定义版型设计空间覆盖率——以标准版型教材（如Aldrich, Metric Pattern Cutting）中的品类清单为参考，统计DSL能表达的品类占比。
