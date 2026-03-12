<!-- markdownlint-disable -->
# Idea 06: 动态适体性评估网络（FitNet）

## 概述

训练一个快速神经代理模型（neural surrogate），从版型几何+身体参数直接预测穿着质量分数（合体性、舒适度、活动自由度），无需运行完整的物理仿真。FitNet可以作为可微奖励函数嵌入版型生成/优化循环中，实现真正的闭环版型设计——在毫秒级内评估版型质量，取代分钟级的物理仿真。

## 指向的Gap

- **Gap 28: 真实穿着舒适度评估的缺失** — 所有AI方法的评估指标都是纯几何或感知层面的，不评估穿着质量
- **Gap 23: 仿真-现实差距** — 通过学习真实穿着数据（而非仅仿真数据）可以部分弥补sim-to-real gap
- **Gap 26: 生成-仿真闭环的缺失** — FitNet作为快速评估器使闭环优化在计算上可行

## 推导过程

### 从Gap到Idea的逻辑链

1. **闭环的核心障碍是速度**：DiffAvatar需要20-200分钟完成一次仿真优化。即使仿真本身是可微的，循环优化仍然太慢。
2. **神经代理的思路**：在分子动力学、流体力学等领域，神经网络已被用作物理模拟的快速代理（surrogate model），加速优化循环数个数量级。
3. **版型域的适用性**：版型→穿着效果的映射虽然复杂，但数据可以通过物理仿真大量生成。训练数据的获取不是瓶颈。

## 详细设计

### 1. 训练数据生成

大规模仿真数据管道：

```
FOR each garment_type in [shirt, pants, dress, jacket, ...]:
  FOR each body_shape in sample(SMPL_space, N=500):
    FOR each pattern_variation in perturb(base_pattern, M=20):
      simulated_garment = cloth_simulate(pattern, body_shape)
      fit_metrics = compute_fit_metrics(simulated_garment, body_shape)
      dataset.append(pattern, body_shape, fit_metrics)
```

**Fit指标计算**（从仿真结果中提取）：

| 指标 | 计算方式 | 语义 |
|------|---------|------|
| **Ease分布** | 各部位body-cloth间距 | 各部位的松紧程度 |
| **压力分布** | cloth stress在body表面的投影 | 穿着压迫感 |
| **应变分布** | fabric strain field | 面料拉伸程度 |
| **悬垂系数** | drape area / fabric area | 悬垂美观度 |
| **穿透量** | body-cloth intersections | 不合理穿透（负值表示不合体） |
| **对称性** | 左右ease分布差异 | 版型对称质量 |

**预估数据规模**：5种品类 × 500体型 × 20变体 = 50,000个仿真样本

### 2. FitNet架构

```
输入：
  - 版型表示：所有panel的B-spline控制点坐标 (P × E × 2)
  - 身体参数：SMPL shape coefficients (β ∈ R^10)
  - 面料属性：(stretch, bend_stiffness, weight)

编码器：
  - Pattern Encoder: Graph Neural Network
    - 节点 = panel顶点
    - 边 = panel内边 + 跨panel的stitch连接
  - Body Encoder: MLP on SMPL β
  - Fabric Encoder: MLP on material parameters

融合：
  - Cross-attention: pattern tokens attend to body features
  - 输出: multi-dimensional fit score vector

输出：
  - Ease vector: 各部位ease预测值 (∈ R^K, K=20个身体关键部位)
  - Comfort score: 综合舒适度评分 (∈ [0, 1])
  - Drape quality: 悬垂质量评分 (∈ [0, 1])
  - Violation flags: 穿透/过紧警告 (binary)
```

### 3. 作为闭环优化的奖励函数

```python
def optimize_pattern(initial_pattern, body, target_fit):
    pattern = initial_pattern.clone().requires_grad_(True)
    optimizer = Adam([pattern], lr=0.01)

    for step in range(1000):
        fit_scores = FitNet(pattern, body)
        loss = mse(fit_scores.ease, target_fit.ease) \
             + lambda1 * (1 - fit_scores.comfort) \
             + lambda2 * fit_scores.violations.sum()
        loss.backward()  # 毫秒级梯度计算
        optimizer.step()
        project_constraints(pattern)  # 保持几何合法性

    return pattern
```

**速度对比**：
- FitNet推理：~10ms/样本
- 完整仿真：~60s/样本（不含优化循环）
- **加速比：~6000x**

### 4. 真实数据校准（可选高级特性）

用少量真实穿着数据（压力传感+运动捕捉）校准FitNet：
- 收集10-20个真实样本（版型+身体+真实fit指标）
- 使用domain adaptation或fine-tuning校准仿真训练的FitNet
- 减小sim-to-real gap

## 参考文献分析

### 核心参考

1. **DiffAvatar** (Li et al., CVPR 2024)
   - "Optimization takes about one minute per iteration and total times vary between 20 to 200 minutes"
   - 直接证明了仿真速度是闭环优化的瓶颈
   - 本提案用神经代理解决这个速度问题

2. **SewingLDM** (ICCV 2025)
   - 唯一将body shape作为生成条件的方法
   - 但不评估合体性——本提案的FitNet可以作为SewingLDM的评估器

3. **Neural Cloth Simulation**
   - GarmentNets, HOOD, Neural Cloth等工作展示了用神经网络近似布料仿真的可能性
   - 本提案不预测完整仿真结果，只预测fit指标——更简单的任务，更容易学习

4. **人体工程学文献**
   - 服装压力舒适度评估（pressure mapping）
   - 活动自由度测试（range of motion analysis）
   - 提供了fit指标的定义和度量标准

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 8/10 | 神经代理模型在版型评估领域未被探索；将fit/comfort指标形式化为可学习的预测任务是新思路 |
| **可行性** | 6/10 | 训练数据可通过仿真自动生成；GNN+cross-attention架构成熟；主要挑战是预测精度能否足够高 |
| **影响力** | 8/10 | 如果精度足够，将使闭环版型优化从理论可能变为实际可行，加速6000倍 |
| **清晰度** | 7/10 | 架构和训练管道清晰，但fit指标的精确定义和评估标准需要更多工作 |
| **证据支撑** | 5/10 | 神经代理模型在其他物理域有效的证据充分，但在版型域无直接先例 |
| **总分** | **34/50** | |

## 风险与挑战

1. **预测精度**：版型→fit的映射高度非线性（微小的版型变化可能导致显著的fit变化），神经网络是否能捕捉这种敏感性存疑
2. **泛化到训练分布外**：如果用户优化出超出训练数据分布的版型，FitNet的预测可能不可靠
3. **仿真数据的偏差**：训练数据来自仿真（而非真实穿着），FitNet继承了仿真的所有偏差
4. **多body shape泛化**：需要在大量不同体型上都保持预测精度
5. **fit指标的主观性**：舒适度在一定程度上是主观的，难以完全形式化
