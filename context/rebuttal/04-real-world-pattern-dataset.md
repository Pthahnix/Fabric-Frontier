<!-- markdownlint-disable -->
# Rebuttal: RealPattern Benchmark Dataset

> Reviewer角色：跨领域审稿人（基准测试方法论 + 数据科学）
> 审阅日期：2026-03-13
> 审阅对象：ideas/04-real-world-pattern-dataset

## 主要反驳意见

### 1. BetterBench的核心教训：质量远比规模重要

BetterBench对现有基准测试的系统性评估揭示了一个残酷事实：MMLU拥有海量数据，但在46条质量标准中仅获得5.5/15——**数据量最大的基准质量最差**。这个教训直接适用于RealPattern：2000-3000个样本的目标数字本身没有意义，关键在于每个样本的标注质量、元数据完整性、版型正确性验证。Idea的评分中Clarity C=9、Evidence E=8，暗示方案已经非常成熟，但如果没有参照BetterBench的46条标准进行系统性设计，这些评分可能过于乐观。一个有500个高质量版型的数据集可能比3000个未经验证的版型更有价值。

### 2. "Can We Trust"的警告：基准测试是规范性工具

基准测试领域的元研究反复警告：benchmark不仅是测量工具，更是"normative instruments"——它们定义了什么是"好的"。对于RealPattern，**谁来定义什么是"好版型"？** 工业生产效率优先的版型（缝份简洁、排料友好）与高级定制优先的版型（合身精确、工艺复杂）有截然不同的评价标准。一个统一的benchmark可能强制推行某种特定的版型美学和工艺标准，边缘化其他同样有价值的传统。Idea需要明确其价值立场，或者设计多维度的评价框架来容纳不同的"好版型"定义。

### 3. FreeSewing数据的代表性偏差

Idea计划从FreeSewing等开源社区获取数据，但这些数据存在严重的系统性偏差。FreeSewing社区以欧美业余裁缝为主，版型风格偏向基础款（T恤、直筒裤），尺码分布以欧美体型为主。这意味着：(a) 缺少复杂结构（如旗袍的连续弯曲、和服的平面构成），(b) 缺少工业级精度的版型（社区版型通常有较大容差），(c) 人体测量数据的种族多样性不足。如果benchmark基于这样的数据，任何在其上训练/测试的模型都会继承这些偏差。

### 4. 版权和知识产权问题被严重低估

真实工业版型是服装企业的核心IP资产。即使是基本的衬衫版型，也承载了企业数十年的试版经验和合身数据。大型服装公司（Zara、Uniqlo、Nike）绝不会公开其版型数据。那么RealPattern的"真实版型"来源只能是：(a) 开源社区（如上所述有偏差），(b) 学术机构自行制作（缺乏工业验证），(c) 逆向工程现成服装（法律风险和精度问题）。每条路径都有严重局限。Idea需要在proposal中坦诚讨论这些数据获取的现实约束。

### 5. 评估指标的定义是最大的学术贡献——但也是最大的挑战

什么是"版型质量"的可计算指标？几何精度（与参考版型的偏差）？缝合兼容性（相邻片的边长匹配度）？排料利用率？穿着合身度（需仿真）？制造可行性（缝制难度）？每个指标都涉及深刻的领域知识和价值判断。Idea如果能定义出一套被社区广泛接受的评估指标体系，这本身就是一篇高影响力论文——但这也意味着benchmark的核心贡献不在数据收集，而在指标设计。Idea的重心应相应调整。

### 6. 版型表示格式的标准化挑战

服装行业使用多种不兼容的版型格式：DXF-AAMA、Gerber格式、Lectra MDL、Optitex PDS、自定义SVG等。RealPattern采用哪种格式？如果选择某种特定格式，就排斥了使用其他格式的社区。如果设计新格式，又面临推广困难。GarmentCode的参数化表示和Sewing Pattern的SVG方案是两种主流学术选择，但都与工业格式不兼容。

### 7. 维护和更新机制

数据集发布后如何演化？发现错误如何修正？新版型如何添加？benchmark的版本管理策略是什么？ImageNet的经验表明：不更新的benchmark会逐渐过时并被overfitting；频繁更新的benchmark则难以横向比较。这个trade-off需要在设计阶段就考虑。

## 改进建议

1. **以BetterBench的46条标准为框架**：逐条审查RealPattern的设计是否满足，将其作为设计规范
2. **分层策略（Complexity Stratification）**：参照SVGen的curriculum设计，将版型分为Basic/Intermediate/Advanced三层，每层有独立的评估标准
3. **多维度评价框架**：不追求单一"好版型"定义，而是提供geometric accuracy、sewing compatibility、nesting efficiency、drape quality等独立指标
4. **小规模高质量起步**：先发布100-200个经过物理仿真验证+专家审核的黄金标准版型，再逐步扩展
5. **版权合规方案**：明确标注每个版型的来源和许可证，建立贡献者协议
6. **活文档（Living Benchmark）设计**：版本号管理，每年更新一次，保留历史版本的可访问性

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 7 | 7 | 维持——服装领域确实缺乏高质量benchmark |
| Feasibility | 7 | 5 | 版权问题、数据质量控制、格式标准化的挑战被低估 |
| Impact | 10 | 9 | 如果质量不高，impact会大打折扣；如果做好，确实是10 |
| Clarity | 9 | 7 | 评估指标体系和数据来源策略讨论不足 |
| Evidence | 8 | 7 | 缺少BetterBench和"Can We Trust"的方法论参考 |

## 补充参考文献

- **BetterBench**: 46条基准质量标准，MMLU仅5.5/15的教训
- **"Can We Trust"系列**: 基准测试作为normative instruments的元批判
- **SVGen**: Complexity stratification的curriculum设计
- **N3D-VLM**: 3D grounding benchmark设计——数值推理的专门评估
- **GarmentCode**: 参数化版型表示方案
- **FreeSewing**: 开源版型社区的数据特征和偏差分析
