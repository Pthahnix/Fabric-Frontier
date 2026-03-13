<!-- markdownlint-disable -->
# Idea 15: 设计意图规格检验（Design Intent Specification Checking）（v1）

## 概述

开发一种**服装设计意图规格语言（Garment Intent Specification Language, GISL）**和分层自动检验引擎。设计师通过自然语言描述设计意图（如"袖子在上臂处应有5cm ease"、"手臂可抬起120度"），系统借助LLM将其翻译为形式化的GISL规格，然后通过分层验证策略——静态约束（廉价）→ 动态约束（昂贵）——自动检查生成的版型是否满足规格。检验结论附带**置信区间**而非二元pass/fail，以反映仿真精度和材料随机性带来的不确定性。

**v1核心修订**：(1) 从"形式化验证(formal verification)"重新定位为"自动规格检验(automated specification checking)"，承认面料物理的随机性使确定性保证不可能；(2) 引入AIDL/SpecifyUI启发的NL→GISL翻译层，降低设计师学习门槛；(3) 区分静态/动态约束的分层验证策略；(4) 讨论generation-time constraint enforcement vs. post-hoc verification的架构权衡；(5) 引入verifier calibration机制。

## 指向的Gap

- **Gap 26: 生成-仿真闭环的缺失** — 当前没有系统能自动评估"版型是否满足设计师意图"
- **Gap 28: 真实穿着舒适度评估的缺失** — 需要将舒适度目标形式化为可度量指标（带置信区间）
- **Gap 14: 从穿着状态逆推静态版型的歧义性** — 形式化的设计目标可以减少逆推的歧义
- **Gap 23: Sim-to-Real Gap** — verifier calibration策略直接针对仿真不可靠性问题

## 推导过程

1. **DiffAvatar优化形状匹配而非设计意图**——它让版型"看起来像"目标3D形状，但不确保穿着者的舒适度或活动自由度。DiffAvatar的target representation是3D shape/appearance；GISL的target representation是可组合、可解释的约束集合——更灵活（可同时指定ease、pressure、ROM、drape等多维约束）且失败时诊断信息更丰富（明确指出哪条约束不满足）
2. **ChatGarment的编辑不一致性**——"altering the skirt length might unintentionally affect other parts"——因为缺乏形式化的不变量保证
3. **AIDL (Jones et al., 2025)的solver-aided DSL范式**——为CAD设计的约束描述DSL证明了"LLM生成约束规格 + external solver验证"的范式可行性。其核心设计决策——语法为LLM可生成性优化、hard/soft constraint分层、约束冲突自动解决——直接可迁移至服装领域。这意味着GISL的新颖性不在于"specification language概念"本身（已非新），而在于**服装领域特定约束体系的形式化定义**（ease、pressure、ROM、drape coefficient等度量的计算语义）
4. **NGL-Prompter的观察**："VLMs struggle to reason about continuous numerical attributes"——自然语言描述设计意图太模糊，需要精确的形式化语言
5. **SpecifyUI (Chen et al., 2025)的设计意图结构化表达**——证明了intermediate representation（SPEC）可以弥合设计师的视觉意图与AI生成之间的鸿沟。用户研究显示结构化规格比纯prompt在intent alignment和controllability上显著更优
6. **nl2postcondition (Endres et al., 2023)的NL→formal spec翻译**——LLM可以将自然语言描述翻译为形式化的method postconditions，且生成的postconditions能捕获64个真实历史bug。这为GISL的NL→spec翻译层提供了直接的方法论支撑
7. **面料物理的随机性**——Kawabata Evaluation System (KES)的16参数体系显示，同一卷面料不同位置的弹性系数可能差5-10%，同一款面料不同批次的手感也会变化。在随机系统上做验证最多只能给出统计保证——"formal verification"的术语过度承诺了系统能力

## 详细设计

### 1. Garment Intent Specification Language (GISL)

**v1重新定位**：GISL的新颖性不在于通用的specification language概念（AIDL已展示），而在于**服装领域特定约束体系的计算定义**。

**约束分类**（v1核心新增——静态/动态分层）：

| 约束类别 | 验证方式 | 计算成本 | 示例 |
|----------|----------|----------|------|
| **几何约束 (G)** | 解析计算（无需仿真） | O(1), ~毫秒 | 版片面积、缝线长度匹配、经线角度 |
| **静态悬垂约束 (S)** | 单次drape simulation | O(N²), ~秒级 | ease分布、悬垂系数、轮廓形状 |
| **动态运动约束 (D)** | 带body motion的biomechanical simulation | O(N³·T), ~分钟至小时 | arm_raise ROM、最大压力、运动限制 |

**声明式规格语法**：

```gisl
garment WomenBlouse {
  // ===== 几何约束 (G) — 解析验证，无需仿真 =====
  geometry.grain_line(front) = vertical ± 2deg
  geometry.seam_length_match(side_front, side_back) <= 3mm

  // ===== 静态悬垂约束 (S) — 单次drape仿真 =====
  static.ease(bust) = 5cm ± 1cm        // target ± tolerance
  static.ease(waist) = 8cm ± 2cm
  static.ease(hip) = 6cm ± 1cm
  static.hem_circumference >= waist_circumference * 1.5
  static.shoulder_point = acromion ± 5mm
  static.dart(front).apex_distance_from_bust = 2cm ± 0.5cm
  static.drape_coefficient >= 0.6
  static.symmetry_deviation <= 2mm

  // ===== 动态运动约束 (D) — biomechanical仿真 =====
  dynamic.arm_raise >= 120deg           // ROM约束
  dynamic.max_pressure(shoulder) <= 0.5 kPa
  dynamic.seated_comfort(waist) = no_restriction

  // ===== 约束元信息 =====
  meta.material_uncertainty = "woven_cotton"  // 查KES数据库获取σ
  meta.confidence_target = 0.95               // 95%置信度
  meta.priority = [static.ease, dynamic.arm_raise, static.drape_coefficient]
}
```

**与AIDL的关键差异**：
- AIDL的约束是几何关系（coincident、parallel、symmetric），通过constraint solver直接求解
- GISL的约束涉及**物理仿真结果**（ease需要布料包裹人体后测量，ROM需要动态仿真），不能通过代数求解器直接验证
- GISL需要处理**材料随机性**——同一约束在不同面料批次上的满足概率不同

### 2. NL→GISL翻译层（v1新增）

**问题**：v0要求设计师直接编写DIL语法，学习曲线过高——SpecifyUI的用户研究确认"设计师难以用语言描述视觉意图"。

**解决方案**：借鉴nl2postcondition (Endres et al., 2023)和AIDL的范式，构建LLM驱动的NL→GISL翻译器：

```
设计师输入（多模态）：
  "我想要一件合体的衬衫，胸围不要太紧，
   手臂能自由抬起，面料要有垂坠感"
  + 参考图片（可选）

LLM翻译：
  → static.ease(bust) = 5cm ± 2cm       // "合体但不太紧" → 中等ease
  → static.ease(waist) = 6cm ± 2cm
  → dynamic.arm_raise >= 100deg          // "手臂自由抬起" → ROM约束
  → static.drape_coefficient >= 0.55     // "垂坠感" → 悬垂系数

设计师确认/修改：
  → 图形化界面显示翻译结果
  → 设计师可调整数值范围（slider）
  → 确认后生成final GISL spec
```

**翻译质量保证**：
- **模板库**：预定义常见服装类型的GISL模板（衬衫、裤装、外套等），LLM在模板基础上修改
- **约束合理性检查**：翻译后自动检查约束之间是否存在逻辑冲突（如ease过小 + ROM过大）
- **参考数据库**：服装工程标准文献中的典型数值范围（如ISO 8559-2:2017 body measurement standards）

### 3. 分层验证引擎（v1核心重设计）

**v0→v1变更**：从统一的"仿真→度量→检查"循环，转向成本感知的分层验证策略。

```
输入：GISL规格 + 候选版型 + 身体模型 + 材料参数

===== 第1层：几何验证（G约束）——解析，~毫秒 =====
for each G_spec in GISL.geometry:
    actual = analytical_compute(pattern)
    result[G_spec] = check(actual, G_spec.target, G_spec.tolerance)
if any G_spec FAILS → 提前终止，返回几何层失败报告

===== 第2层：静态悬垂验证（S约束）——单次仿真，~秒 =====
sim_result = drape_simulate(pattern, body, material)
for each S_spec in GISL.static:
    actual = extract_static_metric(sim_result, body)
    // Monte Carlo估计置信区间（材料参数抖动）
    values = [extract_static_metric(
        drape_simulate(pattern, body, perturb(material, σ_KES)), body
    ) for _ in range(N_mc)]
    ci = confidence_interval(values, level=meta.confidence_target)
    result[S_spec] = {
        point_estimate: actual,
        ci_lower: ci.lower,
        ci_upper: ci.upper,
        status: PASS_CONFIDENT / PASS_MARGINAL / FAIL
    }
if all S_spec PASS_CONFIDENT → 进入第3层
if any S_spec FAIL → 返回静态层失败报告（含偏差量和置信区间）

===== 第3层：动态运动验证（D约束）——仅对最终候选，~分钟 =====
for each D_spec in GISL.dynamic:
    motion_result = biomechanical_simulate(pattern, body, material, motion)
    values = [biomechanical_simulate(..., perturb(material)) for _ in range(N_mc)]
    ci = confidence_interval(values, level=meta.confidence_target)
    result[D_spec] = {point_estimate, ci_lower, ci_upper, status}
```

**计算成本分析**：

| 验证层 | 单次成本 | Monte Carlo (N=20) | 闭环优化 (50迭代) |
|--------|----------|--------------------|--------------------|
| G层 | ~1ms | N/A（确定性） | ~50ms |
| S层 | ~5s | ~100s | ~2.5hr（可微优化可降至~30min） |
| D层 | ~30min | ~10hr | 不可行→仅终验 |

### 4. Verifier Calibration机制（v1新增）

**问题**：Rebuttal核心批评——"verified but wrong"风险。仿真说"arm_raise >= 120deg"满足，但真实穿着中仅能做到100deg，形成false positive。

**Calibration策略**：

```
1. 历史校准数据库：
   sim_pred[ease_bust] = 5.2cm → real_measured = 4.8cm → bias = -0.4cm
   sim_pred[arm_raise] = 125deg → real_measured = 110deg → bias = -15deg
   ...

2. 校准函数：
   real_estimate = sim_value + bias(metric_type, garment_category)
   real_ci = sim_ci * inflation_factor(metric_type)

   其中 inflation_factor > 1.0 反映仿真的系统性乐观偏差

3. 校准状态标注：
   - CALIBRATED: 有≥10个同类型历史校准数据点
   - PARTIALLY_CALIBRATED: 有3-9个数据点，置信区间较宽
   - UNCALIBRATED: 无历史数据，标注为"仿真估计，未经物理验证"
```

### 5. Generation-Time Enforcement vs. Post-Hoc Verification（v1新增）

**问题**：Rebuttal引用FALCON (2025)指出grammar-constrained decoding可实现100% constraint feasibility——如果DIL规格可编码为grammar constraints，则可在**生成时**强制满足约束，无需post-hoc验证循环。

**架构对比分析**：

| 维度 | Generation-Time (FALCON-style) | Post-Hoc (GISL Verifier) |
|------|-------------------------------|--------------------------|
| **适用约束** | 语法/结构约束（版片数量、缝线连接拓扑） | 物理/几何约束（ease、ROM、drape） |
| **保证强度** | 100%（约束嵌入生成过程） | 统计保证（依赖仿真精度） |
| **计算成本** | 生成时额外~20%开销 | 每次验证需要完整仿真 |
| **约束类型限制** | 仅CFG可编码的约束 | 任意可度量的约束 |
| **反馈粒度** | 无（直接满足） | 精确的偏差量+置信区间 |

**结论——互补而非替代**：

- **G约束的子集**（如版片数量、缝线拓扑、经线方向量化）可编码为grammar constraints，在生成时通过constrained decoding强制满足——采用SynCode/DOMINO-style方法
- **S约束和D约束**涉及连续物理量，无法通过grammar constraints编码——必须通过post-hoc仿真验证
- **混合架构**：generation-time enforcement（结构约束）+ post-hoc verification（物理约束），兼具两者优势

### 6. 闭环优化（v1修订）

当验证失败时，利用**DiffCloth-style可微仿真**将验证结果作为梯度信号优化版型：

```
pattern = initial_pattern
for iteration in range(max_iter):
    // 第1层：几何约束（解析梯度）
    g_loss = sum(g_violation ** 2 for g in G_constraints if g.failed)

    // 第2层：静态约束（可微仿真梯度）
    sim_result = differentiable_drape(pattern, body, material)
    s_loss = sum(s_violation ** 2 for s in S_constraints if s.failed)

    // 总损失（加权，按优先级）
    loss = w_g * g_loss + w_s * s_loss
    loss.backward()  // DiffCloth提供端到端梯度
    pattern = pattern - lr * pattern.grad
    pattern = project_to_feasible(pattern)

    if all G+S constraints PASS:
        // 仅在此时运行昂贵的D层验证
        d_result = biomechanical_verify(pattern, body, material)
        if all D constraints PASS:
            return pattern, full_verification_report
        else:
            // D约束失败→调整S约束目标（如增大ease以改善ROM）
            adjust_static_targets(d_result.violations)
```

**关键改进**（相对v0）：
- 闭环优化**仅在G+S层**运行——避免D层的天文数字计算成本
- D层失败时不直接优化D约束，而是**间接调整S约束目标**——利用先验知识（如更大ease通常改善ROM）

### 7. 人机协作工作流（v1修订）

```
设计师 → 自然语言描述意图（+ 参考图片可选）
    ↓ LLM NL→GISL翻译
系统 → 生成GISL规格草案
    ↓
设计师 → 图形化界面审查/修改GISL（slider调整数值）
    ↓ 确认
系统 → 生成候选版型（generation-time enforcement: 结构约束）
    ↓
系统 → 分层验证: G层→S层→D层
    ↓ 报告（含置信区间 + calibration状态）
    ↓ FAIL (S层)
系统 → 可微闭环优化 → 新版型 → 重新验证（G+S）
    ↓ PASS (G+S) but FAIL (D)
系统 → 调整S层目标 → 重新优化
    ↓ PASS 或 达到迭代上限
设计师 → 审查完整报告（每条规格的point estimate + CI + calibration状态）
    ↓ 决策
输出 → 规格检验合格的版型（附验证报告 + 置信度标注）
```

## 参考文献分析

### 直接相关

1. **AIDL** (Jones et al., 2025): Solver-aided DSL for LLM-driven CAD design。将空间推理从LLM卸载给constraint solver，语法为LLM可生成性优化，hard/soft constraint分层。**DIL/GISL的直接先行范式——证明"LLM + constraint spec + solver"架构可行**。关键差异：AIDL的约束是几何关系（可代数求解），GISL的约束涉及物理仿真结果（需要仿真器作为oracle）
2. **SpecifyUI** (Chen et al., 2025): Structured specification for UI design intent。SPEC intermediate representation实现了intent alignment和controllability的显著提升（用户研究N=16）。**支持GISL作为intermediate representation的价值——结构化规格优于纯自然语言prompt**
3. **nl2postcondition** (Endres et al., 2023, 60 citations): LLM将自然语言翻译为形式化postconditions，捕获64个真实bug。**GISL NL翻译层的方法论基础**
4. **DiffAvatar** (Li et al., CVPR 2024): 通过可微仿真优化版型参数以匹配目标3D形状。**GISL闭环优化的技术基础，但target representation不同（3D shape vs. formal spec）**
5. **DiffCloth** (Li et al., 2021, 127 citations): Differentiable cloth simulation with dry frictional contact，85x speedup over naive backpropagation。**验证引擎闭环优化的仿真后端，提供端到端梯度**
6. **Inverse Garment and Pattern Modeling** (Yu et al., 2024): 基于可微仿真的target geometry → pattern optimization。**验证了differentiable simulation在garment inverse design中的端到端可行性**

### 间接相关

7. **Evaluating LLM-Driven User-Intent Formalization** (Lahiri, 2024, 12 citations): 提出specification quality的自动化评估metric——correctness和completeness。**GISL翻译质量评估的方法论参考**
8. **SynCode** (Ugare et al., 2024, 44 citations): Grammar-augmented LLM generation，通过DFA mask store实现语法约束的100% satisfaction。**Generation-time enforcement的技术参考——结构约束可通过constrained decoding强制满足**
9. **DOMINO** (Beurer-Kellner et al., 2024, 109 citations): Fast non-invasive constrained generation for LLMs。**Generation-time constraint enforcement的效率优化参考**
10. **ChatGarment** (CVPR 2025): "altering the skirt length might unintentionally affect other parts"——**GISL不变量保证的动机案例**
11. **Kawabata Evaluation System (KES)**: 面料力学16参数体系——**材料随机性使deterministic verification不可能，支持统计保证+置信区间的设计**
12. **GVU Theory** (2025): "Strengthen the verifier not the generator"——**理论支持verifier价值，但Hallucination Barrier警告specification gaming风险**

### 方法论参考

13. **FALCON** (2025): Grammar-Constrained Decoding + Feasibility Repair Layer实现100% constraint feasibility——**generation-time enforcement的上界参考**
14. **Dress-1-to-3** (Xuan Li et al., 2025): Single image → simulation-ready 3D outfit with differentiable physics。**展示了differentiable garment simulation在实际pipeline中的可集成性**

## 五维评分

| 维度 | v0分数 | v1分数 | 理由 |
|------|--------|--------|------|
| **新颖性 (N)** | 9/10 | 7/10 | AIDL已展示constraint DSL + solver范式（降分），DiffAvatar已做target-driven optimization（降分）。但GISL重新定位到**服装领域特定约束体系的计算定义**（ease/pressure/ROM/drape的形式化度量语义）仍具domain novelty；NL→GISL翻译层+分层验证+calibration机制组合在服装领域无先例 |
| **可行性 (F)** | 4/10 | 5/10 | v1通过分层验证策略显著降低了计算成本（D层仅终验），DiffCloth提供了可微仿真后端，NL翻译层降低用户门槛。但仿真精度（sim-to-real gap）仍是根本瓶颈——calibration只是缓解而非解决 |
| **影响力 (I)** | 9/10 | 9/10 | GVU理论直接支持"strengthen the verifier"方向；如果成功，将实现从"生成并祈祷"到"规格驱动的自动化设计"的范式转变。维持高分 |
| **清晰度 (C)** | 6/10 | 7/10 | 静态/动态约束分层明确化，术语从误导性的"formal verification"修正为"specification checking"，NL翻译层和calibration机制使系统工作流更完整。但generation-time vs. post-hoc的混合架构增加了复杂度 |
| **证据支撑 (E)** | 4/10 | 6/10 | 新增AIDL、SpecifyUI、nl2postcondition、DiffCloth、SynCode、DOMINO等直接/间接相关工作；DiffCloth在garment inverse design的实际应用验证了可微仿真闭环优化的可行性；nl2postcondition的64 bug detection验证了NL→formal spec翻译的实用性 |
| **总分** | **32/50** | **34/50** | 术语修正+分层验证+NL翻译+calibration使设计更务实，证据基础显著增强 |

## 风险与挑战

1. **Sim-to-Real Gap仍是根本瓶颈**：Calibration只能缓解、不能消除。如果仿真系统性地低估某类约束（如肩部压力），即使校准后置信区间也可能过于乐观。需要持续的物理验证数据积累
2. **NL→GISL翻译的歧义性**：自然语言"合体"、"宽松"、"垂坠感"在不同文化/风格语境下含义差异大。翻译器需要contextual understanding——可能需要few-shot examples或设计师profile
3. **多目标约束冲突**：某些GISL规格可能相互矛盾（更小ease vs 更大ROM），需要优先级机制和Pareto front分析
4. **D层约束的计算成本**：即使仅作终验，biomechanical simulation（arm raise from 0→120deg）的单次成本仍可能达30分钟。Monte Carlo置信区间估计需要20次仿真→10小时——对设计迭代周期不可接受
5. **Specification Gaming风险**：GVU Theory警告——如果优化器过度适配verifier的特定仿真行为（而非真实物理），可能找到"technically passing but physically wrong"的解。需要多仿真器cross-validation
6. **约束体系的完备性**：设计师的意图可能包含难以形式化的美学判断（"优雅的廓形"）。GISL只能覆盖可量化的约束子集

## v0→v1 变更说明

| 变更项 | v0 | v1 | 变更原因 |
|--------|----|----|----------|
| **术语定位** | "形式化验证(formal verification)" | "自动规格检验(automated specification checking)" | Rebuttal指出面料物理是随机系统，确定性保证不可能；"formal verification"过度承诺系统能力 |
| **语言名称** | DIL (Design Intent Language) | GISL (Garment Intent Specification Language) | 重新定位新颖性：从通用specification概念转向服装领域特定约束体系 |
| **用户接口** | 设计师直接编写DIL语法 | NL→GISL LLM翻译层 + 图形化审查 | Rebuttal/SpecifyUI指出设计师难以用形式语言描述意图；AIDL/nl2postcondition证明LLM翻译可行 |
| **约束分类** | 未区分静态/动态 | 三层分类：G(几何)/S(静态悬垂)/D(动态运动) | Rebuttal指出静态/动态验证成本差数个数量级，混合使用导致系统不可行 |
| **验证策略** | 每次迭代全约束验证 | 分层验证：G→S→D，D层仅终验 | 动态约束单次~30min，闭环优化中每次验证不可行 |
| **验证结论** | 二元pass/fail | point estimate + 置信区间 + calibration状态 | Rebuttal指出材料随机性和sim-to-real gap使确定性判断不可靠；需要统计保证 |
| **Calibration** | 无 | 历史校准数据库 + bias correction + inflation factor | Rebuttal指出"verified but wrong"风险——verifier必须经过物理验证校准 |
| **Generation-time** | 未讨论 | 讨论FALCON-style enforcement vs. post-hoc verification | Rebuttal指出generation-time constraint enforcement可能优于post-hoc循环；v1分析表明两者互补 |
| **与DiffAvatar对比** | 简单提及 | 详细论证GISL vs. 3D target representation的trade-off | Rebuttal要求论证为什么formal spec比3D target更好 |
| **新颖性评分** | N=9 | N=7 | AIDL已展示constraint DSL+solver范式，DiffAvatar已做target-driven optimization |
| **可行性评分** | F=4 | F=5 | 分层验证降低计算成本，DiffCloth提供可微后端，NL翻译降低用户门槛 |
| **清晰度评分** | C=6 | C=7 | 约束分层明确化，术语修正，NL翻译层和calibration使工作流完整 |
| **证据评分** | E=4 | E=6 | 新增AIDL/SpecifyUI/nl2postcondition/DiffCloth/SynCode/DOMINO等10+篇直接相关工作 |
| **总分** | 32/50 | 34/50 | +2分，反映术语务实化+设计完善+证据增强 |
| **参考文献** | 5篇（无直接先行工作） | 14篇（6篇直接相关 + 4篇间接相关 + 4篇方法论参考） | 大幅增强证据基础，覆盖DSL设计、NL翻译、可微仿真、constrained decoding等关键技术维度 |
