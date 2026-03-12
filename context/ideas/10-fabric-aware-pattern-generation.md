<!-- markdownlint-disable -->
# Idea 10: 面料感知版型生成

## 概述

将面料材料属性作为**一等公民输入**（first-class input）集成到版型生成模型中。同一款式使用不同面料时，版型应该自动调整——丝绸衬衫 vs 牛仔衬衫需要不同的松量(ease)、不同的缝份宽度、不同的省道设计、不同的grain line方向。本提案通过面料属性编码器+条件生成机制实现面料感知的版型生成。

## 指向的Gap

- **Gap 19: 面料约束的缺失** — AI生成版型时完全不考虑面料的物理属性和使用约束
- **Gap 13: 非刚性面料的展平失真** — 弹性面料需要负ease，当前方法不处理
- **Gap 23: 仿真-现实差距** — 面料感知版型生成可减少仿真与实际的偏差

## 推导过程

1. **AIpparel明确指出**：limitations中将"consideration of physical and material constraints during sewing pattern prediction"列为future work
2. **实际制版实践**：使用丝绸vs牛仔布做同一款衬衫，版型的ease量、省道设计、缝份宽度都不同
3. **方法论**：在现有生成模型（如GarmentDiffusion的DiT）的conditioning机制中加入面料属性向量

## 详细设计

### 1. 面料属性编码

定义标准化的面料属性向量：

| 属性 | 范围 | 数据来源 |
|------|------|---------|
| 重量(weight) | 50-500 g/m² | 纺织品规格表 |
| 经向弹性(warp_stretch) | 0-50% | ASTM D5035 |
| 纬向弹性(weft_stretch) | 0-50% | ASTM D5035 |
| 弯曲刚度(bend_stiffness) | 0.01-10 N·mm | ASTM D1388 |
| 悬垂系数(drape_coeff) | 0.2-0.9 | ISO 9073 |
| 厚度(thickness) | 0.1-5 mm | 直接测量 |
| 面料幅宽(width) | 90-160 cm | 规格表 |
| 表面摩擦系数 | 0.1-0.8 | ASTM D1894 |
| 面料类型(one-hot) | [woven, knit, nonwoven] | 分类 |

**面料编码器**：
- MLP: fabric_vector (9维+分类) → fabric_embedding (128维)
- 预训练：在面料属性→仿真悬垂效果的预测任务上预训练

### 2. 条件生成机制

以GarmentDiffusion的DiT架构为基础，加入面料条件：

```
原始GarmentDiffusion:
  输入 = [noisy_edge_tokens, text_condition, image_condition, timestep]

面料感知版本:
  输入 = [noisy_edge_tokens, text_condition, image_condition,
          fabric_embedding, timestep]

Fabric conditioning方式:
  - AdaLN（与timestep类似的调制方式）
  - Cross-attention to fabric tokens
```

### 3. 面料驱动的版型适配规则

通过数据学习的隐式规则（也可以与显式知识图谱结合）：

| 面料特性变化 | 版型适配 |
|-------------|---------|
| 高弹性(stretch>20%) | 负ease（版型小于体型10-30%） |
| 高悬垂(drape>0.7) | 减少省道、增加裙摆量 |
| 高刚度(stiff>5 N·mm) | 增加ease、使用立体裁剪 |
| 厚面料(thick>2mm) | 增加缝份宽度 |
| 薄透面料 | 法式缝、包缝（缝份内折） |
| 条纹/格纹 | 需要图案对花约束 |

### 4. 训练数据增强

现有合成数据集不包含面料信息。数据增强策略：
- 为GarmentCodeData中的每个版型模拟多种面料条件下的穿着效果
- 根据穿着效果反推不同面料应有的版型调整量
- 生成(面料, 版型调整)的配对训练数据

## 参考文献分析

1. **AIpparel** (CVPR 2025): Limitations明确将面料约束列为future work
2. **DiffAvatar** (CVPR 2024): 恢复材料参数（弹性/弯曲）但不反馈到版型设计
3. **SewingLDM** (ICCV 2025): Body-shape conditioning但完全忽略面料
4. **纺织工程标准**: ASTM D5035（拉伸）、D1388（弯曲）、ISO 9073（悬垂）提供标准化的面料属性测试方法
5. **实际制版教材**: Aldrich《Metric Pattern Cutting》详细讨论了不同面料的版型调整规则

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 7/10 | 将面料属性作为生成条件的想法直观但无人实现 |
| **可行性** | 6/10 | 条件生成机制成熟，主要挑战在于获取面料属性-版型调整的配对训练数据 |
| **影响力** | 9/10 | 面料是服装设计的核心要素，面料感知版型生成将显著提升实际可用性 |
| **清晰度** | 8/10 | 属性定义、编码方式、conditioning机制都明确 |
| **证据支撑** | 7/10 | AIpparel/SewingLDM已展示conditioning机制；制版教材提供了面料-版型关系知识 |
| **总分** | **37/50** | |

## 风险与挑战

1. **面料属性获取**：用户可能不知道面料的精确物理属性，需要从面料名称/照片推断
2. **面料-版型关系的复杂性**：同一面料的不同批次可能有不同属性
3. **训练数据缺乏**：当前没有(面料属性, 版型调整)的配对数据集
4. **评估困难**：需要真实面料+版型的实物验证
