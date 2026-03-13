<!-- markdownlint-disable -->
# Idea 12: 图案对花约束求解器（v1）

## 概述

开发一个专门处理**图案对花（Pattern Matching/Alignment）**问题的混合约束求解系统。当使用条纹、格纹、印花等有规律图案的面料时，版片在面料上的放置位置必须保证图案在缝合线两侧对齐——这是工业制版中最困难的约束满足问题之一。当前没有任何AI方法涉及这一问题。

**v1核心修订**：放弃以MIP（Mixed Integer Programming）为主求解器的架构，转向**Diffusion-based Gradient Field + MIP验证层**的混合方案；引入制造不确定性建模（tolerance band）和quality-cost Pareto前沿分析；明确benchmark数据集构建路径。

## 指向的Gap

- **Gap 19: 面料约束的缺失——图案对花** — 条纹/格纹面料需要跨版片图案连续，这是极难的约束满足问题
- **Gap 20: 排料优化** — 图案对花约束显著影响排料效率

## 推导过程

1. **工业痛点**：使用格纹面料时，面料浪费率可从常规的15-25%飙升至30-50%，因为版片放置位置受到图案重复单元的严格限制
2. **计算复杂度**：图案对花本质上是在2D排列+图案相位约束下的组合优化问题，MIP在工业规模（N=20-50片）下面临变量空间O(N×M×P)和二次增长的约束数量，branch-and-bound搜索难以在可接受时间内收敛
3. **Diffusion-based方法的突破**：GFPack++ (Xue et al., 2024) 证明score-based diffusion model可以从teacher packing examples中学习gradient field，在garment数据上达到甚至超越teacher algorithm的空间利用率，且支持连续旋转和任意边界泛化——这为图案对花约束的可微编码提供了可行路径
4. **组合式约束求解**：Diffusion-CCSP (Yang et al., 2023) 展示了将约束满足问题分解为独立的diffusion model（每个model学习一种约束类型），然后通过能量组合实现全局约束满足——图案对花可以作为一种新的约束类型插入此框架
5. **制造不确定性**：Textile IR (2026) 报告的compound uncertainty ~26%揭示了服装制造中不确定性的普遍性，图案对花必须从确定性约束升级为带容差的stochastic约束

## 详细设计

### 1. 图案表示

将面料图案建模为2D周期性函数：

```
Print(x, y) = F(x mod Tx, y mod Ty, θ)

其中：
- Tx, Ty: 水平/垂直重复周期
- θ: 图案旋转角度
- F: 图案函数（图像或参数化描述）
```

| 图案类型 | Tx | Ty | θ | 约束难度 |
|----------|----|----|---|---------|
| 横条纹 | ∞ | Ty | 0° | 低——只需水平对齐 |
| 竖条纹 | Tx | ∞ | 0° | 低——只需垂直对齐 |
| 格纹（对称） | Tx | Ty | 0° | 中——需要双向对齐 |
| 格纹（非对称） | Tx | Ty | 0° | 高——需要方向匹配 |
| 斜纹 | Tx | Ty | 45° | 极高——旋转约束 |
| 自由印花 | Tx | Ty | any | 极高——完整2D相位匹配 |

### 2. 对花约束形式化（含Tolerance Band）

对于每对缝合在一起的边 (eᵢ, eⱼ)：

**确定性约束（理想情况）**：
```
Print(Panel_i(eᵢ(t))) = Print(Panel_j(eⱼ(t))), ∀t ∈ [0, 1]
```

**v1修订——带容差的stochastic约束**：
```
|Phase_i(t) - Phase_j(t)| ≤ τ(quality_level)

其中：
- Phase_i(t) = (xᵢ(t) mod Tx, yᵢ(t) mod Ty)
- τ(quality_level): 对花容差函数
  - Haute Couture: τ < 1mm
  - Commercial Ready-to-Wear: τ < 3mm
  - Fast Fashion: τ < 5mm（或不约束）
- 面料印刷误差 ε ~ N(0, σ²_print), σ_print ≈ 0.3% × repeat
- 裁剪/缝制拉伸 δ ~ U(-0.5mm, +0.5mm)
```

### 3. 混合求解架构（v1核心重设计）

**v0→v1变更**：从"MIP为主"转向"Diffusion Gradient Field + MIP验证"。

#### 3.1 主求解引擎——Diffusion-based Gradient Field

借鉴GFPack++ (Xue et al., 2024)和Diffusion-CCSP (Yang et al., 2023)的方法：

**约束分解与组合**：
```
E_total = E_no_overlap + E_pattern_align + E_boundary + E_grain_line

其中：
- E_no_overlap: 无重叠约束（标准packing约束，GFPack++已实现）
- E_pattern_align: 图案对花约束（新增——核心贡献）
- E_boundary: 面料幅宽约束
- E_grain_line: 经纬方向约束
```

**图案对花能量函数**：
```
E_pattern_align = Σ_{(i,j)∈Seams} ∫₀¹ w(t) · d_phase(Panel_i(eᵢ(t)), Panel_j(eⱼ(t))) dt

其中：
- d_phase: 周期性相位距离（考虑mod Tx, Ty的环面距离）
- w(t): 权重函数——缝合线可见区域权重高，隐藏区域（如腋下）权重低
```

**训练流程**：
1. 使用传统heuristic算法（含对花约束）生成teacher examples
2. 训练attention-based score network学习gradient field
3. Inference时通过iterative refinement生成排料方案

#### 3.2 MIP验证层

MIP不再作为主求解器，而是作为**feasibility checker**：

```
给定Diffusion输出的排列方案 S*：
  1. 验证所有对花约束是否在tolerance τ内
  2. 验证无重叠约束（NFP-based精确验证）
  3. 若违反，标记违反区域 → 反馈给Diffusion做局部修正
  4. 输出feasibility report（满足/违反的约束列表）
```

参考FALCON (2026)的Grammar-Constrained Decoding + Feasibility Repair Layer理念：生成层（Diffusion）负责效率，验证层（MIP/NFP）保证100%可行性。

#### 3.3 Quality-Cost Pareto分析（v1新增）

```
对于给定面料和版型集合：
  SWEEP τ from τ_min (1mm) to τ_max (5mm):
    SOLVE packing with tolerance τ
    RECORD (fabric_utilization(τ), pattern_matching_quality(τ))

  输出：
  - Pareto前沿曲线：对花精度 vs 面料利用率
  - 各quality_level下的最优排料方案
  - 边际分析：放松1mm容差可节省多少面料
```

### 4. 与版型生成的集成

**v1增强——端到端可微集成**：

```
路径A（后处理，v0设计保留）：
  版型生成（任何方法） → 版片集合
    ↓
  面料选择（格纹/条纹面料） → 图案参数
    ↓
  图案对花求解器 → 优化排列方案（含Pareto分析）
    ↓
  输出：排料图（含对花标记 + tolerance report）

路径B（端到端，v1新增）：
  版型生成模型 ←→ 对花约束梯度
    ↓
  联合优化：让生成模型直接输出"对花友好"的版型
  （缝合线形状适配图案周期，减少对花难度）
```

路径B的核心insight来自rebuttal：如果pattern generation模型是可微的（如SewingLDM (Cao et al., 2026)使用latent flow matching），那么对花约束的梯度可以反向传播到生成模型，使其学会避免产生难以对花的缝合线形状。

### 5. Benchmark数据集构建（v1新增）

**动机**：E=4的核心原因是缺乏evaluation infrastructure。

**数据集设计**：

| 数据集 | 规模 | 图案类型 | 复杂度 |
|--------|------|----------|--------|
| PatternMatch-Easy | 100例 | 横/竖条纹 | 1D对齐 |
| PatternMatch-Medium | 200例 | 对称格纹 | 2D对齐 |
| PatternMatch-Hard | 100例 | 非对称格纹/斜纹 | 2D+旋转 |
| PatternMatch-Real | 50例 | 真实工业扫描 | 含印刷误差 |

**数据来源**：
1. 合成数据：从SewFactory/GarmentCodeData版型 + 参数化图案生成
2. 工业数据：与面料供应商合作获取真实repeat尺寸和误差分布
3. 评估指标：
   - Phase Error (mm): 缝合线处的图案相位偏差
   - Fabric Utilization (%): 面料利用率
   - Solve Time (s): 求解时间
   - Pareto Dominance: 多目标优化质量

### 6. 多层对花的分层处理（v1新增）

**问题**：外层面料与里布的格纹对齐产生组合爆炸。

**解决策略——分层解耦**：
```
Layer 1 (外层面料): 完整对花约束
Layer 2 (里布): 仅在可见区域（领口、下摆翻折处）施加对花约束
Layer 3 (衬布): 无对花约束

约束传递：
  外层排料方案 → 固定可见区域相位 → 约束里布排料
  （单向传播，避免双向耦合的组合爆炸）
```

参考Dress-1-to-3的CIPC differentiable framework处理多层布料物理交互的思路，但在图案对花场景下采用分层解耦策略降低复杂度。

## 参考文献分析

### 直接相关（v0保留 + v1新增）

1. **无AI先行工作**：据调研，没有任何学术论文用AI解决图案对花问题——学术空白确认
2. **Gerber/Lectra排料工具**：工业CAD有基本的条纹/格纹模式，但优化能力有限，无法处理复杂图案
3. **2D nesting文献** (Wascher et al., 2007): 标准nesting不考虑图案约束
4. **GFPack++** (Xue et al., 2024): Attention-based gradient field learning for 2D irregular packing，支持连续旋转和任意边界泛化，在garment数据集上超越teacher algorithm。**直接适用作为主求解引擎**
5. **Learning-Based 2D Packing** (Yang et al., 2023): RL-based hierarchical packing pipeline，5-10% improvement over baselines，domain transfer从一般packing到garment-specific packing可行
6. **Diffusion-CCSP** (Yang et al., 2023): Compositional diffusion models for continuous constraint satisfaction，通过能量组合实现多约束联合满足——**图案对花可作为新约束类型插入此框架**

### 间接相关（v1新增）

7. **Textile IR** (2026): Compound uncertainty ~26% in textile manufacturing，seven-layer Verification Ladder，**支持uncertainty建模的必要性**
8. **FALCON** (2026): Grammar-Constrained Decoding + Feasibility Repair Layer实现100% constraint satisfaction feasibility，**验证层设计参考**
9. **Dress-1-to-3** (2025): CIPC differentiable framework处理multi-layer cloth interaction，**多层对花场景参考**
10. **Computational Pattern Making** (Pietroni et al., 2022): 3D→2D版型生成中显式处理seam symmetry、grain alignment等制造约束，**缝合线约束形式化参考**
11. **SewingLDM** (Cao et al., 2026): Latent flow matching for sewing pattern generation，**端到端集成路径B的基础**
12. **Minkowski Penalties** (SIGGRAPH): Robust differentiable constraint enforcement for vector graphics，**可微约束编码的数学工具参考**

## 五维评分

| 维度 | v0分数 | v1分数 | 理由 |
|------|--------|--------|------|
| **新颖性 (N)** | 9/10 | 9/10 | 图案对花的计算化在学术界仍完全空白，不调整 |
| **可行性 (F)** | 5/10 | 6/10 | v0的MIP路径确实可行性堪忧（rebuttal指出F应降至4）；但v1转向diffusion-based方案后，GFPack++已在garment packing上验证有效，加入图案约束的技术路径更清晰，回升至6 |
| **影响力 (I)** | 7/10 | 7/10 | 工业价值真实但仅限有图案面料——保持不变 |
| **清晰度 (C)** | 7/10 | 8/10 | v1补充了Pareto分析、tolerance band建模、benchmark设计、多层策略，设计完整度显著提升 |
| **证据支撑 (E)** | 4/10 | 5/10 | GFPack++在garment packing的实证、Diffusion-CCSP的约束组合范式、FALCON的feasibility guarantee为技术路径提供了间接但有力的证据支撑；benchmark数据集构建计划弥补了evaluation infrastructure缺失 |
| **总分** | **32/50** | **35/50** | 技术路径重设计+补充设计使得整体质量提升 |

## 风险与挑战

1. **图案约束的可微编码**：将离散的图案相位匹配转化为连续可微的能量函数是核心技术难点——周期性mod运算的梯度不连续性需要smooth approximation（如soft-mod函数）
2. **Teacher example生成**：Diffusion方法依赖高质量teacher packing examples，但目前不存在处理图案对花的heuristic algorithm——需要先开发基线heuristic
3. **图案信息获取**：用户需要准确描述面料图案的重复单元，可能需要computer vision辅助（从面料照片自动提取Tx, Ty, θ）
4. **曲线缝合线的对花**：直线缝合线的对花相对简单，曲线缝合线（如袖窿）的相位距离计算需要沿曲线积分，计算成本高
5. **实用性范围**：仅限有规律图案的面料，对素色面料无意义
6. **端到端集成路径（路径B）的稳定性**：将对花梯度反传到pattern generation模型可能导致训练不稳定，需要careful gradient scaling

## v0→v1 变更说明

| 变更项 | v0 | v1 | 变更原因 |
|--------|----|----|----------|
| **主求解架构** | MIP为主求解器 | Diffusion Gradient Field为主 + MIP为验证层 | Rebuttal指出MIP在工业规模下计算不可行；GFPack++已证明diffusion-based packing在garment数据上有效 |
| **约束建模** | 确定性硬约束 | 带tolerance band的stochastic约束 | Rebuttal指出面料印刷误差（~0.3% repeat）和裁剪拉伸使确定性建模不切实际；Textile IR报告26% compound uncertainty |
| **质量-成本权衡** | 未讨论 | 新增Quality-Cost Pareto前沿分析 | Rebuttal指出不同产品层次的对花标准差异巨大，需要参数化的tradeoff |
| **评估基础** | 无benchmark | 新增PatternMatch-{Easy/Medium/Hard/Real}数据集设计 | Rebuttal核心批评E=4源于research infrastructure缺失 |
| **多层对花** | 未讨论 | 新增分层解耦策略 | Rebuttal指出多层对花的组合爆炸问题 |
| **端到端集成** | 仅后处理 | 新增路径B——可微端到端集成 | Rebuttal建议让生成模型学会输出"对花友好"的版型 |
| **参考文献** | 5篇（无直接AI工作） | 12篇（6篇直接相关 + 6篇间接相关） | 新增GFPack++、Diffusion-CCSP、FALCON、Textile IR、Dress-1-to-3、SewingLDM等关键引用 |
| **可行性评分** | F=5 | F=6 | 技术路径从unproven MIP转向有garment packing实证的diffusion方法 |
| **清晰度评分** | C=7 | C=8 | 补充了完整的系统设计（tolerance、Pareto、benchmark、多层策略） |
| **证据评分** | E=4 | E=5 | 间接证据显著增强（GFPack++、Diffusion-CCSP、FALCON），benchmark构建计划弥补infrastructure gap |
| **总分** | 32/50 | 35/50 | +3分，反映技术路径重设计带来的整体改善 |
