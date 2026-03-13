<!-- markdownlint-disable -->
# Idea 04: 真实世界版型数据集（RealPattern Benchmark）（v1）

## 概述

构建首个**质量优先、多维评价**的真实世界版型基准数据集——RealPattern Benchmark。v0版本以"首期2000-3000个真实版型"为目标，重心放在数据规模上。v1版本吸收了BetterBench 46条质量标准的系统性警告、FreeSewing代表性偏差批评、版权合规挑战、以及"benchmark作为normative instrument"的元批判，将设计重心从**规模导向**转向**质量导向 + 指标体系设计**。核心变更：(1) 采用"小规模黄金标准 → 逐步扩展"的分阶段策略（Phase 1: 200个经过物理仿真验证+专家审核的高质量版型），(2) 引入基于BetterBench框架的系统性质量保证协议，(3) 设计多维评价框架（Multi-Perspective Evaluation Framework）容纳不同"好版型"定义，(4) 明确版权合规方案和数据来源偏差缓解策略。

## 指向的Gap

- **Gap 5: 数据集偏差——合成 vs 真实** — 所有训练数据来自程序化合成（GarmentCode引擎），存在根本性分布偏移。NGL-Prompter (2026) 进一步证实："prior approaches fine-tune on synthetic garment datasets...often struggle to generalize to in-the-wild images"，且GarmentCode随机参数采样产生不真实的组合（如不对称上衣的左右部分不协调）
- **Gap 27: 评估指标的碎片化** — 每篇论文用不同数据集和指标，缺乏公平比较基准。BetterBench (NeurIPS 2024) 揭示MMLU等大规模benchmark在46条质量标准中仅获5.5/15——规模不等于质量
- **Gap 33: Benchmark作为Normative Instrument的价值立场缺失** — 工业生产效率优先 vs 高级定制合身度优先 vs 文化传统版型保护，需要多维评价框架而非单一标准

## 推导过程

### 从Gap到Idea的逻辑链

1. **核心观察**：DressWild (2026) 和 NGL-Prompter (2026) 均明确承认合成数据泛化问题。NGL-Prompter进一步指出GarmentCode的随机参数采样"often yields unrealistic or inconsistent garments"——问题不仅是合成vs真实的分布偏移，更在于合成数据的内部质量
2. **BetterBench的教训**（Reuel et al., NeurIPS 2024）：对24个AI benchmark的系统评估发现，MMLU拥有海量数据但质量评分极低。46条最佳实践覆盖benchmark全生命周期（设计→实施→文档→维护）。这直接否定了v0"首期2000-3000个版型"的规模导向策略
3. **4D-DRESS的方法论启示**（Wang et al., CVPR 2024）：作为首个真实世界4D服装数据集，其"semi-automatic parsing pipeline + human-in-the-loop"方法论证明了高质量真实数据的可行路径——96.8%的帧无需人工干预，但剩余帧中仅1.5%的顶点需要修正。关键在于pipeline设计而非人力堆积
4. **Benchmark的normative本质**：基准测试领域的元研究反复警告benchmark不仅是测量工具，更是"normative instruments"——它们定义了什么是"好的"。对于版型领域，工业效率、定制合身、文化传统三种价值观不可被单一标准覆盖
5. **数据来源的现实约束**：FreeSewing以欧美业余裁缝为主，缺少旗袍/和服等复杂结构；工业版型是商业核心IP。v1必须坦诚面对这些约束并设计缓解策略

## 详细设计

### 1. 分阶段数据收集策略（Quality-First Phasing）

#### Phase 1: 黄金标准集（Gold Standard Set）——200个版型

目标：建立质量标杆和标注规范，而非追求规模。

| 来源 | 规模 | 特点 | 质量保证 |
|------|------|------|---------|
| **FreeSewing.org** | 40-50 | 参数化JavaScript，可精确提取几何 | 自动化验证+专家审核 |
| **Seamly2D/Valentina** | 30-40 | 开源CAD，.val/.sm2d格式 | CAD导出精度验证 |
| **时尚院校合作**（教学版型） | 60-80 | 经过教学验证的经典版型 | 物理制作+仿真验证双重确认 |
| **独立版型设计师** | 30-40 | 付费收集，签署贡献者协议 | 专家pair review |
| **非西方传统版型** | 20-30 | 旗袍/和服/韩服/印度Saree等 | 文化领域专家审核 |

**每个版型的质量验证流程**：
```
Step 1: 格式解析与几何验证
  → 边长匹配检查（相邻panel缝合边长度误差 < 2mm）
  → 闭合性检查（每条缝合边都有配对）
  → grain line合理性检查

Step 2: 物理仿真验证
  → 使用ContourCraft/XPBD模拟器进行3D draping
  → 检查穿透(penetration)、过度拉伸(overstretching)
  → 仿真成功率要求 > 95%

Step 3: 专家审核（每个版型至少2位独立审核人）
  → 审核维度: 几何正确性、缝合可行性、工艺合理性
  → 不一致时由第3位审核人仲裁
  → 审核记录保留作为元数据
```

#### Phase 2: 扩展集（Extended Set）——500-800个版型

在Phase 1建立的标注规范和质量协议基础上扩展：
- 引入半自动化标注pipeline（参考4D-DRESS的方法论）
- 降低人工审核强度（抽检20%而非全量审核）
- 扩展数据来源至历史版型档案和更多国际合作

#### Phase 3: 社区贡献集（Community Set）——持续增长

- 开放贡献者投稿通道，提供标准化模板和验证工具
- 社区贡献的版型经过自动验证后标记为"community-verified"，区别于"expert-verified"
- 版本号管理（年度发布，保留历史版本可访问性）

### 2. 数据标准化格式

每个版型样本包含：

```json
{
  "id": "RP-00001",
  "version": "1.0",
  "source": {
    "origin": "fashion_academy_X",
    "contributor": "...",
    "license": "CC-BY-SA-4.0",
    "original_format": "DXF-AAMA",
    "acquisition_method": "CAD_export"
  },
  "verification": {
    "tier": "expert-verified",
    "reviewers": 2,
    "simulation_passed": true,
    "seam_compatibility_score": 0.98,
    "review_date": "2026-05-01"
  },
  "garment_type": "women_blouse",
  "cultural_context": "western_contemporary",
  "complexity_tier": "intermediate",
  "panels": [
    {
      "name": "front_bodice",
      "geometry": {
        "format": "B-spline",
        "control_points": [...],
        "degree": 3
      },
      "seam_allowance": {...},
      "notches": [...],
      "grain_line": {...}
    }
  ],
  "stitches": [...],
  "fabric_spec": {
    "type": "cotton_poplin",
    "weight_gsm": 120,
    "width_cm": 150,
    "stretch_percent": {"warp": 2, "weft": 5}
  },
  "size_spec": {
    "system": "EU",
    "base_size": 38,
    "measurements": {...}
  },
  "annotations": {
    "difficulty": "intermediate",
    "construction_notes": "...",
    "text_description_en": "...",
    "text_description_zh": "...",
    "rendered_images": {
      "front": "...",
      "back": "...",
      "detail": "..."
    }
  }
}
```

**格式选择决策**：采用B-spline控制点作为规范几何表示，同时提供格式转换工具链：
- DXF-AAMA → B-spline（工业格式导入）
- Gerber/Lectra → B-spline（工业格式导入）
- GarmentCode parametric → B-spline（学术格式桥接）
- B-spline → SVG（可视化和社区分发）

### 3. 多维评价框架（Multi-Perspective Evaluation Framework）

**核心改进（回应"benchmark是normative instrument"批评）**：不追求单一"好版型"定义，而是提供**四个独立评价维度**，每个维度对应不同的利益相关方和价值观。

#### 维度 A: 几何精度（Geometric Fidelity）
面向**学术研究者**和**方法比较**

| 指标 | 说明 | 计算方式 |
|------|------|---------|
| Panel Chamfer Distance (PCD) | panel形状精度 | 采样点集间Chamfer距离 |
| Edge Curve Error (ECE) | 边曲线拟合误差 | 对应边的Hausdorff距离 |
| Panel Count Accuracy (PCA) | panel数量预测准确率 | 精确匹配率 |
| Topology F1 (TF1) | 缝合关系正确率 | 预测缝合对的F1分数 |

#### 维度 B: 制造可行性（Manufacturability）
面向**工业用户**和**生产效率**

| 指标 | 说明 | 计算方式 |
|------|------|---------|
| Seam Compatibility Score (SCS) | 相邻panel缝合边长度匹配度 | 1 - |Δlength|/avg_length |
| Seam Allowance Correctness (SAC) | 缝份添加正确性 | 与参考值的MAE |
| Notch Completeness (NC) | 标记点完整性 | 检测率 |
| Nesting Utilization (NU) | 排料利用率 | 优化排料后的面料利用率 |

#### 维度 C: 穿着质量（Wearability）
面向**设计师**和**合身度评估**

| 指标 | 说明 | 计算方式 |
|------|------|---------|
| Simulation Drape Error (SDE) | 仿真悬垂差异 | 3D Chamfer距离（draping后） |
| Fit Error on Body (FEB) | 合体性偏差 | 关键围度的MAE |
| Ease Distribution (ED) | 松量分布合理性 | 与参考ease的KL散度 |
| CLIP Similarity (CS) | 渲染图视觉相似度 | CLIP embedding余弦相似度 |

#### 维度 D: 文化多样性（Cultural Diversity）
面向**公平性**和**全球代表性**

| 指标 | 说明 | 计算方式 |
|------|------|---------|
| Cultural Coverage (CC) | 文化/区域覆盖度 | 非西方版型占比 |
| Body Diversity (BD) | 体型多样性 | 尺码分布的entropy |
| Style Diversity (SD) | 风格多样性 | 版型聚类分析的diversity指数 |

**关键设计决策**：四个维度独立报告，**不合并为单一总分**。不同应用场景的用户可以根据自身需求选择关注的维度组合。

### 4. 数据偏差缓解策略

#### 4.1 FreeSewing偏差问题的系统性应对

| 偏差类型 | 表现 | 缓解措施 |
|---------|------|---------|
| 风格偏差 | 偏向基础款（T恤、直筒裤） | Phase 1即引入非西方传统版型（20-30个） |
| 精度偏差 | 社区版型有较大容差 | 仅接受通过物理仿真验证的版型 |
| 体型偏差 | 欧美体型为主 | 要求尺码分布覆盖至少3个人体测量标准（EU/US/CN） |
| 复杂度偏差 | 缺少复杂结构 | 设定复杂度配额：Simple ≤35%, Medium 35-45%, Complex ≥20% |

#### 4.2 版权合规方案

```
版权分级制度:
  Level A: 完全开放 (CC-BY-SA / MIT)
    → FreeSewing、Seamly2D、自制教学版型
    → 可自由用于训练和评估

  Level B: 仅评估使用 (Evaluation-Only License)
    → 工业合作伙伴提供的脱敏版型
    → 可用于benchmark评估，不可用于训练

  Level C: 受控访问 (Controlled Access)
    → 敏感来源版型（逆向工程验证后的公开款式）
    → 需签署数据使用协议后访问

每个版型附带明确的许可证标注和来源追溯链
贡献者协议(CLA)明确知识产权归属和使用范围
```

### 5. Baseline实现与BetterBench合规

为所有主要范式提供参考实现：
- **Autoregressive**: SewingGPT baseline
- **Diffusion**: GarmentDiffusion baseline
- **Program Synthesis**: Design2GarmentCode baseline
- **Training-Free**: NGL-Prompter baseline（v1新增——代表最新的zero-shot方法）
- **Image-based**: Sewformer / GarmageNet baseline

**BetterBench合规检查清单**（覆盖46条标准中与版型benchmark相关的核心条目）：

| BetterBench维度 | 关键条目 | RealPattern对应措施 |
|----------------|---------|-------------------|
| **设计** | 领域专家参与 | 版型师+仿真工程师双审核 |
| **设计** | 明确范围声明 | 声明四个评价维度和适用边界 |
| **实施** | 评估脚本公开 | 开源evaluation toolkit |
| **实施** | 统计显著性报告 | 要求bootstrap置信区间 |
| **文档** | 数据卡(Datasheet) | 遵循Datasheets for Datasets模板 |
| **文档** | 许可证明确 | 三级版权分级制度 |
| **维护** | 反馈通道 | GitHub Issues + 年度社区会议 |
| **维护** | 版本管理 | 语义化版本号，年度发布 |

### 6. Living Benchmark设计

```
版本管理策略:
  v1.0 (Phase 1): 200个expert-verified版型，四维评价框架
  v1.x (增量更新): bug修复，标注修正，不影响已发布排行
  v2.0 (Phase 2): 500-800个版型，扩展数据源
  v3.0 (Phase 3): 社区贡献集纳入

  原则:
  - 每个major version保留独立排行榜
  - Test set不公开ground truth（仅通过evaluation server提交）
  - 年度错误修正窗口，保留修正日志
  - 退役标准参考BetterBench的benchmark deprecation框架
```

## 参考文献分析

### 核心参考

1. **BetterBench** (Reuel et al., NeurIPS 2024)
   - 46条AI benchmark质量标准，覆盖设计→实施→文档→维护全生命周期
   - MMLU海量数据但质量评分极低的教训——直接否定规模导向策略
   - RealPattern的质量保证协议以此为框架

2. **4D-DRESS** (Wang et al., CVPR 2024)
   - 首个真实世界4D服装数据集（64 outfits, 78K scans）
   - Semi-automatic parsing pipeline + human-in-the-loop方法论
   - 96.8%帧无需人工干预——证明高质量真实数据的可行路径
   - 为RealPattern的半自动标注流程提供方法论参考

3. **NGL-Prompter** (Badalyan et al., 2026)
   - Training-free sewing pattern estimation，新收集~5000张in-the-wild图像（ASOS数据集）
   - 明确指出GarmentCode随机采样"often yields unrealistic or inconsistent garments"
   - 证实了对真实世界版型-图像配对数据的迫切需求

4. **GarmentCodeData** (Korosteleva et al., ECCV 2024)
   - 最大的版型数据集（115K），但完全合成
   - 其数据格式和自动draping pipeline可作为技术参考
   - 其局限性（合成分布偏移）是RealPattern存在的根本动机

5. **SewFactory** (Liu et al., SIGGRAPH Asia 2023)
   - ~1M图像+sewing pattern的合成数据集
   - Sewformer的训练数据——代表当前合成数据的最佳实践
   - 与RealPattern互补而非替代关系

6. **How2Bench** (Cao et al., 2025)
   - 274个代码相关benchmark的系统审查，提出55条检查清单
   - 与BetterBench互补，强调benchmark的可复现性和统计严谨性

7. **Deep Fashion3D** (Zhu et al., ECCV 2020)
   - 2078个真实服装3D模型，10个类别
   - 从真实服装重建3D模型的先例——但缺少sewing pattern标注
   - 潜在的cross-modal对齐数据源

### 对比参考

8. **GarmentLab** (Lu et al., 2024)
   - 服装manipulation的统一benchmark
   - 其sim-to-real算法和real-world benchmark组件的设计可借鉴
   - 但关注manipulation而非pattern generation

9. **ImageNet** (Deng et al., CVPR 2009)
   - 标准化基准改变CV领域的先例
   - v1更谨慎地类比：ImageNet的成功不仅在规模，更在其持续的社区维护和竞赛机制

## 五维评分

| 维度 | v0分 | v1分 | 理由 |
|------|------|------|------|
| **新颖性** | 7/10 | 7/10 | 维持——版型领域确实缺乏高质量benchmark，首次系统性设计 |
| **可行性** | 7/10 | 6/10 | 下调——版权问题、数据质量控制、格式标准化的工程挑战被v0低估。Phase 1(200个)可行，但扩展到Phase 3有显著不确定性 |
| **影响力** | 10/10 | 9/10 | 微调——如果Phase 1的200个高质量版型+四维评价框架做好，影响力接近10；但规模限制可能降低训练数据价值 |
| **清晰度** | 9/10 | 8/10 | 下调——v1虽然改善了评估指标设计，但多维评价框架的社区接受度存在不确定性；格式转换工具链的技术细节需进一步明确 |
| **证据支撑** | 8/10 | 8/10 | 维持——新增BetterBench框架、4D-DRESS方法论、NGL-Prompter的实证支持。但版型领域的真实数据收集仍缺少直接先例 |
| **总分** | **41/50** | **38/50** | 评分更诚实——反映了工程实施的真实难度 |

## 风险与挑战

1. **版权和商业敏感性**（风险等级：高）：真实工业版型是企业核心IP。v1通过三级版权分级制度缓解，但Level B/C的数据获取仍依赖商业谈判
2. **数据格式多样性**（风险等级：中）：DXF-AAMA/Gerber/Lectra等格式的解析需要大量工程工作。v1通过B-spline统一表示+格式转换工具链应对
3. **标注一致性**（风险等级：中）：不同来源的版型标注规范不同。v1通过Phase 1建立标准化协议后再扩展
4. **FreeSewing代表性偏差**（风险等级：中）：社区数据以欧美基础款为主。v1通过复杂度配额和文化多样性维度缓解
5. **多维评价框架的社区接受度**（风险等级：中）：四维独立评价可能不如单一排行榜直观，社区采用需要推广
6. **规模与质量的trade-off**（风险等级：低-中）：Phase 1仅200个版型，作为训练数据价值有限。需明确定位为evaluation benchmark而非training dataset——训练仍依赖合成数据，但evaluation在真实分布上进行
7. **Living benchmark的维护成本**（风险等级：中）：年度更新、评估服务器维护需要持续投入

## v0→v1 变更说明

| 变更项 | v0 | v1 | 变更原因 |
|--------|----|----|---------|
| **数据规模策略** | 首期2000-3000个版型 | Phase 1: 200个黄金标准 → Phase 2: 500-800 → Phase 3: 社区扩展 | BetterBench教训：MMLU规模大但质量评分极低。200个expert-verified版型比3000个未验证版型更有价值 |
| **质量保证机制** | 未明确 | 三步验证流程（格式验证→物理仿真→专家双审核）| 回应rebuttal"质量远比规模重要"的核心批评 |
| **评价框架** | 10个指标的单一列表 | 四维独立评价框架（几何精度/制造可行性/穿着质量/文化多样性）| 回应"benchmark是normative instrument"警告——不同用户有不同的"好版型"定义 |
| **数据偏差处理** | 未讨论 | 系统性偏差缓解策略（复杂度配额、文化多样性要求、体型覆盖标准）| 回应FreeSewing代表性偏差批评 |
| **版权方案** | "需要明确授权"（一句话） | 三级版权分级制度（开放/仅评估/受控访问）+ 贡献者协议 | 回应"版权问题被严重低估"批评 |
| **数据格式** | 仅描述JSON schema | B-spline统一表示 + 多格式转换工具链 | 回应格式标准化挑战 |
| **Baseline** | 4个范式 | 5个范式（新增NGL-Prompter作为training-free代表）| 反映2026年最新方法进展 |
| **维护策略** | 未涉及 | Living Benchmark设计（语义化版本号、年度发布、退役标准）| 回应"维护和更新机制"批评 |
| **BetterBench合规** | 未参考 | 系统性对标46条标准，提供合规检查清单 | 回应rebuttal的核心方法论建议 |
| **评分调整** | 总分41/50 | 总分38/50 | 更诚实地反映工程实施难度，特别是可行性(7→6)和影响力(10→9)的下调 |
| **新增参考文献** | — | BetterBench, 4D-DRESS, NGL-Prompter, How2Bench, Deep Fashion3D | 系统性补充benchmark方法论和真实数据集的文献支撑 |
