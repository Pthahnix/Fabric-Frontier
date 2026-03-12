<!-- markdownlint-disable -->
# Idea 09: GarmentBench 统一评估基准

## 概述

构建GarmentBench——AI版型生成领域的首个**统一评估基准**，类比NLP中的GLUE/SuperGLUE和CV中的ImageNet。GarmentBench包含标准化测试集、统一指标套件、标准评估协议和主要范式的baseline实现，使得自回归、扩散、程序合成、图像化等不同范式能够在同一舞台上公平比较。

## 指向的Gap

- **Gap 9: 范式间缺乏公平对比** — 四种根本不同的生成范式无法公平比较
- **Gap 27: 评估指标碎片化** — 每篇论文使用不同指标和数据集
- **Gap 5: 数据集偏差** — 提供多来源数据减轻单一合成引擎的偏差

## 推导过程

1. **观察**：SewingGPT用Panel L2，AIpparel用CLIP score，ChatGarment用user study——指标不可比
2. **NLP先例**：GLUE benchmark (Wang et al., 2019) 通过统一9个NLP任务的评估推动了BERT→GPT→T5的快速迭代
3. **版型领域的特殊性**：版型评估需要**多层级指标**——几何精度、拓扑正确性、物理可行性、穿着质量、可制造性——这比NLP更复杂

## 详细设计

### 1. 测试集构成

**规模**：500+ 服装样本，按复杂度和品类分层

| 复杂度 | Panel数 | 样本数 | 品类示例 |
|--------|---------|--------|---------|
| Simple | ≤6 | 150 | A字裙、T恤、直筒裤 |
| Medium | 7-12 | 200 | 衬衫、连衣裙、休闲裤 |
| Complex | 13+ | 150 | 夹克、大衣、套装 |

**多模态输入**：每个样本配备
- 文本描述（专业版型术语 + 日常语言两个版本）
- 3D渲染图（正面、侧面、背面）
- 技术草图（flat sketch）
- 设计参数表

**Ground Truth**：
- 精确的panel几何（B-spline控制点）
- 完整的stitch关系
- 缝份宽度
- 标记点位置
- Grain line方向
- 3D仿真渲染参考

### 2. 统一指标套件

**Tier 1: 几何准确性**（最基本，所有方法都应报告）
- **Panel Chamfer Distance (PCD)**: 预测panel与GT panel的Chamfer距离，归一化
- **Edge Curve Error (ECE)**: 边曲线的参数化拟合误差
- **Panel Count Accuracy (PCA)**: panel数量预测准确率

**Tier 2: 拓扑正确性**
- **Stitch F1 (SF1)**: 缝合关系的F1 score
- **Topology Graph Edit Distance (TGED)**: 预测的panel-stitch图与GT图的编辑距离

**Tier 3: 物理可行性**
- **Simulation Success Rate (SSR)**: 版型能否成功仿真到身体上（无穿透、无发散）
- **Drape Similarity (DS)**: 仿真悬垂效果与GT的相似度

**Tier 4: 可制造性**（高级指标，鼓励但不强制）
- **Seam Allowance Score (SAS)**: 自动添加缝份的正确性
- **DXF Export Score (DES)**: 能否导出有效DXF文件

**Tier 5: 感知质量**
- **CLIP Similarity (CS)**: 渲染图与GT渲染图的CLIP余弦相似度
- **Human Preference Score (HPS)**: 人类盲审偏好评分

### 3. 评估协议

- 固定数据划分（不允许在测试集上调参）
- 所有方法在相同硬件上报告推理时间
- 在线排行榜（blind test set, 防止overfit）
- 年度更新测试集（防止test set overfitting）

### 4. Baseline实现

为四种主要范式提供PyTorch参考实现：

| 范式 | Baseline | 实现基础 |
|------|---------|---------|
| 自回归 | SewingGPT-base | 基于DressCode论文复现 |
| 扩散 | GarmentDiff-base | 基于GarmentDiffusion论文复现 |
| 程序合成 | GarmentCode-gen | 基于Design2GarmentCode简化版 |
| 图像化 | Garmage-base | 基于GarmageNet概念 |

## 参考文献分析

### 核心参考

1. **GLUE Benchmark** (Wang et al., 2019)
   - 统一了9个NLP任务的评估
   - 催生了BERT → RoBERTa → T5 → GPT-3的快速迭代
   - GarmentBench借鉴其多任务评估和在线排行榜设计

2. **SewingGPT/DressCode** (He et al., SIGGRAPH 2024)
   - 定义了Panel L2、Stitch F1等指标
   - GarmentBench将其纳入Tier 1/2并标准化

3. **GarmentDiffusion** (Li et al., IJCAI 2025)
   - 声称复现SewingGPT进行比较，但复现质量不透明
   - GarmentBench通过提供标准baseline消除复现偏差

4. **AIpparel** (Nakayama et al., CVPR 2025)
   - 使用CLIP score和FID——完全不同的指标体系
   - GarmentBench将CLIP归入Tier 5，与几何指标共存

5. **ChatGarment** (Bian et al., CVPR 2025)
   - 依赖user study评估——主观性强
   - GarmentBench将HPS纳入Tier 5但要求同时报告客观指标

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 7/10 | 基准构建本身不是方法创新，但分层指标设计（从几何到可制造性）是新的 |
| **可行性** | 8/10 | 数据可从现有合成数据集+少量真实数据构建；评估协议可以明确定义；baseline可以复现 |
| **影响力** | 10/10 | 将改变整个领域的评估方式，推动公平比较和快速迭代——标准化基准的影响力是长期的、根本性的 |
| **清晰度** | 9/10 | 指标层级、评估协议、数据划分都可以精确定义 |
| **证据支撑** | 8/10 | GLUE/ImageNet的成功先例；领域对指标碎片化的广泛认知 |
| **总分** | **42/50** | |

## 风险与挑战

1. **社区采纳**：基准的价值取决于社区是否广泛使用，需要与顶会workshop合作推广
2. **复杂度vs可用性**：5层指标可能过于复杂，需要合理的"必选+可选"划分
3. **数据质量**：GT版型的精度直接影响评估的公正性，需要严格的质量控制
4. **范式差异**：不同范式的输出格式差异很大（程序 vs token序列 vs 图像），统一评估接口需要仔细设计
5. **维护成本**：基准需要持续维护和更新
