<!-- markdownlint-disable -->
# Rebuttal: GarmentCode++ Self-Learning DSL

> Reviewer角色：跨领域审稿人（程序合成 + 服装CAD）
> 审阅日期：2026-03-13
> 审阅对象：ideas/01-garmentcode-plus-plus-self-learning-dsl

## 主要反驳意见

### 1. DreamCoder引用严重过时，LILO才是正确的技术基线

Idea以DreamCoder作为library learning的核心参考，但LILO（ICLR 2024）已经在多个方面全面超越DreamCoder：速度提升1000-10000倍，库质量提升+42.27（相对于base DSL）。更关键的是，LILO发现了一个对本Idea生死攸关的现象：**AutoDoc机制至关重要**。匿名抽象（anonymous abstractions）不仅无用，反而导致性能下降-30.60。这意味着如果GarmentCode++学到的版型原语没有语义命名（例如"袖山曲线"而非"lambda_27"），整个library learning pipeline将失效。Idea必须以LILO为基线重新设计，并将AutoDoc——即自动为学到的抽象生成人类可读语义描述——作为核心模块而非可选组件。

### 2. Hallucination Barrier：自生成+自评估=停滞

GVU理论（2025）提出了一个根本性警告：**当同一个系统既生成输出又评判输出质量时，self-improvement会在某个点停滞**。这正是GarmentCode++面临的风险——如果LLM生成版型DSL代码，同时也由LLM判断版型质量，则验证器的signal-to-noise ratio不会高于生成器，改进将触顶。GVU的核心处方是"Strengthen the verifier, not the generator"。对于服装版型，**外部验证器必须是物理仿真**（如DiffCloth，85x speedup使其实际可用），而非LLM自身的判断。Idea中完全缺失这一层external verification机制，这是架构层面的缺陷。

### 3. AIDL提供了更务实的替代路径

AIDL（2025）的核心洞察是：与其让现有DSL自学习扩展，不如**从头为LLM设计一种新的DSL**。AIDL为图形系统构建了"tailored for language models"的DSL，并配合constraint solver将空间推理从LLM中卸载。这一思路直接适用于服装版型：与其在GarmentCode的现有语法上做library learning，不如设计一种LLM-native的版型描述语言，将几何约束交给求解器处理。CAD-LLaMA的hierarchical description（准确率从39.17%跃升至80.41%）进一步证明：描述语言的层次结构设计对LLM性能有决定性影响。Idea应当至少讨论这条替代路径，并论证为何library learning优于重新设计。

### 4. 服装领域复杂度远超LILO验证过的领域

LILO在REGEX、LOGO等相对简单的领域验证了library learning的有效性。但服装版型涉及三维几何、材料力学、人体形态学、缝制工艺等多维度约束的交叉——复杂度量级完全不同。Text-to-CadQuery的成功（124M模型超越363M specialist）虽然鼓舞人心，但CadQuery的Python API本身已经非常成熟。GarmentCode的DSL是否具有同等水平的基础设施？如果基础DSL不够表达力，library learning学到的抽象也无法弥补底层的不足。

### 5. SGP-Bench的语义理解数据需要谨慎解读

SGP-Bench显示LLM对symbolic programs有>80%的语义理解能力，但这是在相对标准的程序上测量的。服装DSL的"语义理解"需要跨越代码语义和物理语义的鸿沟——理解"这段代码生成袖山"和理解"这个袖山是否适合目标面料的悬垂性"是两个完全不同的能力层次。QuantiPhy和DeepPHY的发现更令人警惕：VLM在物理推理上"do not reason but memorize"，CoT在21个任务中19个反而损害表现。

### 6. Self-Improving Coding Agent的类比需要限定

Self-Improving Coding Agent（17%→53% SWE-Bench）和Darwin Godel Machine（20%→50%）的成功确实暗示自改进范式可行，但两者都有明确的外部验证信号（测试通过/不通过）。服装版型的"测试"是什么？仿真结果的好坏判定本身就需要领域专业知识。Idea需要定义清晰的、可自动化评估的质量指标。

## 改进建议

1. **替换技术基线**：从DreamCoder迁移至LILO，将AutoDoc作为一等模块
2. **引入物理仿真验证层**：DiffCloth或SDRS作为external verifier，构建"生成→仿真→反馈"闭环
3. **与AIDL路径做对比实验**：在proposal中设计ablation——library learning vs. LLM-native DSL redesign
4. **定义可自动化评估的质量指标**：如仿真后的应力分布、碰撞检测通过率、缝合边匹配度等
5. **增加语义命名验证实验**：对比有/无AutoDoc的library learning效果，量化语义命名的贡献

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 9 | 8 | LILO已推进library learning，本Idea并非首次将其引入程序合成 |
| Feasibility | 5 | 5 | LILO证明library learning可行（↑），但服装复杂度远超已验证领域（↓），两者抵消 |
| Impact | 9 | 9 | 维持——若成功将是变革性的 |
| Clarity | 7 | 6 | 缺失LILO/AIDL讨论、缺失external verifier设计，方案完整性不足 |
| Evidence | 6 | 5 | 核心引用过时，跨领域证据整合不足 |

## 补充参考文献

- **LILO** (ICLR 2024): Library learning，AutoDoc机制，+42.27提升，1000-10000x加速
- **GVU Theory** (2025): Hallucination Barrier，external verifier必要性
- **AIDL** (2025): LLM-native DSL设计，constraint solver卸载空间推理
- **CAD-LLaMA** (2025): Hierarchical description，39.17%→80.41%
- **DiffCloth**: 可微服装仿真，85x speedup
- **QuantiPhy / DeepPHY**: VLM物理推理能力的局限性
- **Self-Improving Coding Agent** (2025): 自改进范式的外部验证信号
