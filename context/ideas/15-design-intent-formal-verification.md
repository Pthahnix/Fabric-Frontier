<!-- markdownlint-disable -->
# Idea 15: 设计意图形式化验证（Design Intent Verification）

## 概述

开发一种**设计意图规格语言（Design Intent Language, DIL）**和自动验证引擎。设计师用DIL描述版型应满足的功能/美学/舒适性目标（如"袖子在上臂处应有5cm ease"、"裙摆周长应为腰围的2倍"），系统自动生成版型、仿真、度量、验证是否满足所有规格，不满足则反馈修改。这实现了从"生成并祈祷"到"规格驱动的自动化设计"的范式转变。

## 指向的Gap

- **Gap 26: 生成-仿真闭环的缺失** — 当前没有系统能评估"版型是否满足设计师意图"
- **Gap 28: 真实穿着舒适度评估的缺失** — 需要将舒适度目标形式化为可度量指标
- **Gap 14: 从穿着状态逆推静态版型的歧义性** — 形式化的设计目标可以减少逆推的歧义

## 推导过程

1. **DiffAvatar优化形状匹配而非设计意图**——它让版型"看起来像"目标3D形状，但不确保穿着者的舒适度或活动自由度
2. **ChatGarment的编辑不一致性**——"altering the skirt length might unintentionally affect other parts"——因为没有形式化的约束来保证不变量
3. **软件工程类比**：形式化验证（formal verification）在软件工程中已被成功应用。版型设计可以类比为"编写满足规格的程序"——规格就是设计意图
4. **NGL-Prompter的观察**："VLMs struggle to reason about continuous numerical attributes"——自然语言描述设计意图太模糊，需要精确的形式化语言

## 详细设计

### 1. Design Intent Language (DIL)

**声明式规格语法**：

```dil
garment WomenBlouse {
  // 合体性规格
  fit.ease(bust) = 5cm ± 1cm
  fit.ease(waist) = 8cm ± 2cm
  fit.ease(hip) = 6cm ± 1cm
  fit.ease(upper_arm) = 3cm ± 1cm

  // 轮廓规格
  silhouette.hem_circumference >= waist_circumference * 1.5
  silhouette.shoulder_point = acromion  // 肩缝落在肩峰点

  // 结构规格
  structure.dart(front).apex_distance_from_bust = 2cm ± 0.5cm
  structure.grain_line(front) = vertical  // 前片经线垂直

  // 舒适度规格
  comfort.arm_raise >= 120deg  // 手臂可抬起120度无限制
  comfort.max_pressure(shoulder) <= 0.5 kPa

  // 美学规格
  aesthetic.drape_coefficient >= 0.6  // 悬垂系数
  aesthetic.symmetry_deviation <= 2mm  // 左右对称偏差
}
```

### 2. 验证引擎

```
输入：DIL规格 + 候选版型 + 身体模型

步骤1：仿真
  simulated_garment = cloth_simulate(pattern, body)

步骤2：度量提取
  measurements = extract_metrics(simulated_garment, body)
  // ease分布、压力分布、ROM、悬垂系数等

步骤3：规格检查
  for each spec in DIL:
    actual_value = measurements[spec.metric]
    if spec.check(actual_value):
      result[spec] = PASS
    else:
      result[spec] = FAIL(actual=actual_value, expected=spec.target)

步骤4：报告
  返回 pass/fail 列表 + 每项的偏差量
```

### 3. 闭环优化

当验证失败时，使用验证结果作为梯度信号优化版型：

```
pattern = initial_pattern
for iteration in range(max_iter):
    sim_result = simulate(pattern, body)
    metrics = extract_metrics(sim_result, body)
    violations = check_specs(metrics, dil_specs)

    if all_pass(violations):
        return pattern  // 所有规格满足

    // 计算损失并反传
    loss = sum(violation.deviation ** 2 for v in violations if v.failed)
    loss.backward()
    pattern = pattern - lr * pattern.grad

    // 物理合法性投影
    pattern = project_to_feasible(pattern)
```

### 4. 人机协作工作流

```
设计师 → 编写DIL规格（图形化界面辅助）
    ↓
系统 → 自动生成候选版型（使用现有AI方法）
    ↓
系统 → 仿真+验证 → 报告pass/fail
    ↓ FAIL
系统 → 自动优化 → 新版型 → 重新验证
    ↓ PASS 或 达到迭代上限
设计师 → 审查结果 → 修改DIL规格或接受
    ↓
输出 → 验证合格的版型
```

## 参考文献分析

1. **形式化验证文献**: Model checking (Clarke et al.)、property specification languages (PSL, SVA) — 方法论基础
2. **DiffAvatar** (CVPR 2024): 优化for形状匹配，而非设计意图。本提案用DIL规格代替3D目标形状作为优化目标
3. **ChatGarment** (CVPR 2025): "altering the skirt length might unintentionally affect other parts"——正因为缺乏形式化的不变量保证
4. **NGL-Prompter** (2026): "VLMs struggle to reason about continuous numerical attributes"——自然语言不够精确，需要形式化
5. **人体工程学**: 压力舒适度阈值、ROM标准——提供了DIL规格的度量标准参考

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 9/10 | 将形式化验证引入版型设计是全新范式；DIL规格语言在服装领域没有先例 |
| **可行性** | 4/10 | DIL语言设计可行，但自动度量提取（从仿真结果中精确测量ease等）和仿真速度是主要障碍 |
| **影响力** | 9/10 | 如果成功，将从根本上改变版型设计方式——从手工调整到规格驱动的自动化 |
| **清晰度** | 6/10 | DIL语法和验证逻辑清晰，但与现有生成模型的集成细节需要进一步研究 |
| **证据支撑** | 4/10 | 形式化验证在软件工程中成功但版型域的物理复杂度更高；无直接先例 |
| **总分** | **32/50** | |

## 风险与挑战

1. **DIL规格的完备性**：设计师的意图可能包含难以形式化的美学判断（"优雅的廓形"无法量化）
2. **仿真精度依赖**：验证结果的可信度取决于仿真精度——sim-to-real gap直接影响验证可靠性
3. **度量提取的困难**：从仿真结果中精确测量"ease"需要自动识别身体landmark和服装对应点
4. **多目标冲突**：某些DIL规格可能相互矛盾（更紧=更贴身 vs 更大ROM），需要优先级机制
5. **学习曲线**：设计师需要学习DIL语法——降低采纳意愿
