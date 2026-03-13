<!-- markdownlint-disable -->
# Rebuttal: Design Intent Formal Verification

> Reviewer角色：跨领域审稿人（formal methods + program synthesis + garment engineering）
> 审阅日期：2026-03-13
> 审阅对象：ideas/15-design-intent-formal-verification

## 主要反驳意见

### 1. AIDL已展示"solver-aided DSL for LLMs"范式——DIL新颖性需重新定位

AIDL（2025）为图形系统设计了一种**专为LLM优化的约束描述DSL**，将空间推理从LLM中卸载给constraint solver。这与DIL的核心思路高度重合——将设计意图形式化为可验证的约束规范。AIDL的关键设计决策直接可迁移：(a) 语法为LLM可生成性优化而非人类可读性优化；(b) hard/soft constraint分层；(c) 约束冲突的自动解决机制。这意味着"constraint specification + solver verification"范式本身不是新的——该Idea的新颖性需要重新定位到**garment domain-specific的约束体系设计**（如ease、pressure、ROM、drape的形式化定义），而非通用的specification language概念。仅在broader program synthesis context中审视，DIL的conceptual contribution是incremental的。

### 2. FALCON证明generation-time约束保证优于post-hoc验证

FALCON（2025）通过**grammar-constrained decoding + repair + sampling**实现了100%的约束可行性。如果DIL的设计意图规范可以编码为grammar constraints，那么FALCON的方法可以在**生成时**就强制满足约束——不需要post-hoc的"生成→仿真→验证→修改"循环。这将DIL从"检查工具"升级为"生成保证"。该Idea目前的架构是先生成pattern再用verifier检查——这在计算效率上远不如generation-time enforcement。FALCON-style的方法可以直接消除不可行解的生成，避免昂贵的仿真验证循环。Idea应讨论这两种架构的trade-off，而非默认采用post-hoc verification。

### 3. 验证引擎的可靠性取决于仿真精度——"verified but wrong"风险

GVU Theory（2025）的"strengthen the verifier"主张支持DIL的方向，但这里存在一个关键前提：**verifier本身必须是可靠的**。DIL的验证引擎依赖物理仿真来检查pattern是否满足spec（如arm_raise >= 120deg），而仿真精度本身就是一个未解决的问题（Gap 23: sim-to-real gap）。如果仿真说"满足comfort.arm_raise >= 120deg"但真实穿着中实际仅能做到100deg，那么formal verification给出的是**false positive**——"verified but wrong"。这比没有验证更危险，因为它给了用户虚假的信心。Idea必须讨论verifier calibration策略：仿真结果如何通过physical validation校准？验证结论应该附带confidence interval而非binary pass/fail。

### 4. 静态与动态验证的计算复杂度差异被忽视

DIL语法中混合了静态约束（如`fit.ease(bust) = 5cm ± 1cm`）和动态约束（如`comfort.arm_raise >= 120deg`）。这两类约束的验证成本相差**数个数量级**。静态约束只需要一次drape simulation（秒级），而动态约束需要带body motion的完整biomechanical simulation——手臂从静止抬升到120度的动态过程涉及布料-身体接触检测、摩擦力建模、惯性效应等，单次仿真可能需要数小时。Idea的closed-loop optimization中如果每次迭代都要验证动态约束，计算成本将使系统完全不可行。需要设计分层验证策略：先用静态约束（廉价）筛选候选方案，再用动态约束（昂贵）验证最终方案。

### 5. "形式验证"类比具有误导性——面料物理是随机系统而非确定系统

该Idea从software formal verification借用了"formal verification"概念，但两者的执行语义有根本差异。软件具有**确定性执行语义**——给定相同输入，程序总是产生相同输出，因此形式验证可以给出数学证明级别的保证。面料物理是**随机系统**——同一卷面料的不同位置弹性系数可能差5-10%，同一款面料不同批次的颜色和手感也会变化。在随机系统上做"形式验证"最多只能给出**统计保证**（如"95%置信区间内满足spec"），不能给出确定性保证。称其为"formal verification"过度承诺了系统能力——更准确的术语是"automated specification checking"或"constraint satisfaction testing"，但这在学术上的新颖性显著降低。

### 6. DiffAvatar已实现"optimize pattern to match target"——DIL的增量贡献需明确

DiffAvatar通过可微渲染和仿真实现了**从target appearance反向优化pattern参数**的能力。这与DIL的closed-loop optimization在精神上非常相似——区别在于target representation：DiffAvatar使用3D shape/appearance作为target，DIL使用formal specification作为target。Idea需要论证为什么formal specification比3D target更好。一个可能的论点是：formal specification更灵活（可以组合多个约束）且更可解释。但反面论点是：大多数设计师更容易用reference image表达意图（"像这件衣服一样"）而非写formal spec——DiffAvatar的target representation对用户更友好。

## 改进建议

1. **重新定位新颖性**：从"specification language概念"转向"garment-specific约束体系的形式化"——ease/pressure/ROM/drape的计算定义才是domain contribution
2. **讨论FALCON-style generation-time enforcement**：与post-hoc verification做架构对比，评估在garment generation场景下哪种更优
3. **设计分层验证策略**：静态约束（廉价，每次迭代都检查）→ 动态约束（昂贵，仅对最终候选检查）
4. **引入verifier calibration**：验证结论附带confidence interval，基于sim-to-real gap的历史数据校准仿真精度
5. **术语修正**：将"formal verification"降级为"automated specification checking"，或者明确讨论在随机系统中formal verification的含义限定
6. **设计natural language到formal spec的翻译层**：与AIDL类比，让设计师用自然语言描述意图，系统翻译为DIL spec

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 9 | 7 | AIDL已展示constraint DSL + solver范式，DiffAvatar已做target-driven optimization，DIL增量有限 |
| Feasibility | 4 | 4 | 动态约束验证的计算成本极高，sim-to-real gap导致verifier不可靠，维持低分 |
| Impact | 9 | 9 | GVU理论直接支持verifier价值，方向正确，维持 |
| Clarity | 6 | 5 | 静态/动态约束混淆、"形式验证"术语误导、target user未定义，清晰度不足 |
| Evidence | 4 | 4 | 未引用AIDL/FALCON/DiffAvatar等直接相关工作，维持低分 |

## 补充参考文献

- **AIDL** (2025): Solver-aided DSL for LLMs, constraint specification + solver verification — DIL的直接先行范式
- **FALCON** (2025): Grammar-constrained decoding, 100% constraint feasibility — generation-time enforcement替代post-hoc verification
- **GVU Theory** (2025): "Strengthen verifier not generator" — 理论支持，但Hallucination Barrier警告specification gaming风险
- **DiffAvatar**: Target-driven pattern optimization via differentiable rendering — DIL的alternative target representation
- **DiffCloth** (ICLR 2023): Differentiable cloth simulation, 85x speedup — 验证引擎的潜在仿真后端
- **Kawabata Evaluation System (KES)**: 面料力学16参数体系 — 材料随机性使deterministic verification不可能
