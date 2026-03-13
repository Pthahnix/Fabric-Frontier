<!-- markdownlint-disable -->
# Rebuttal: FitNet - Dynamic Fit Evaluation Network

> Reviewer角色：跨领域审稿人（neural surrogate + 服装仿真）
> 审阅日期：2026-03-13
> 审阅对象：ideas/06-dynamic-fit-evaluation-network

## 主要反驳意见

### 1. GVU理论同时支持和威胁FitNet

GVU理论（2025）的核心主张"Strengthen the verifier, not the generator"从战略层面直接支持FitNet的价值定位——一个独立的、高质量的fit评估器确实是版型生成pipeline中最有价值的组件。然而，GVU同时揭示了Hallucination Barrier：**如果verifier本身是neural网络（即FitNet），它的signal-to-noise ratio是否足够高？** neural surrogate学到的是仿真数据的统计模式，而非物理规律。当输入超出训练分布（新体型、新面料、极端尺码），surrogate的预测可靠性会急剧下降——这恰恰是fit评估最需要准确的场景。FitNet可能在"简单case上快速给出正确答案"但在"困难case上快速给出错误答案"，后者比完全没有评估更危险。

### 2. 可微仿真的进步可能已经使neural surrogate不必要

DiffCloth提供85x的梯度计算加速。Dress Anyone实现5-30分钟的完整pipeline（从设计到真实制造），包含仿真环节。Inverse Garment达到15x GPU加速。这些进展共同指向一个问题：**物理仿真是否已经快到不需要neural surrogate？** FitNet声称6000x加速，但如果基线仿真已经从"数小时"降到"几分钟"，6000x加速意味着毫秒级——速度固然惊人，但"几分钟"的仿真在大多数工业流程中已经完全可接受。neural surrogate的价值在于大规模优化（需要数千次评估），但在单次设计评估场景中，可微仿真可能已经够快了。

### 3. 50K训练样本的成本被严重低估

FitNet需要大规模仿真数据训练。假设每个样本需要：(a) 一个版型几何，(b) 一个人体模型，(c) 仿真运行，(d) 结果标注。即使使用DiffCloth的加速，每个仿真至少需要数十秒到几分钟。50K样本 × 2分钟/样本 = 约70天的单GPU连续仿真时间。考虑到需要覆盖多种体型、多种面料、多种版型风格的组合空间，50K样本可能严重不足——而如果需要500K样本，训练数据生成本身就成为一个重大工程项目。

### 4. Sim-to-real校准的具体方案缺失

Idea提到sim-to-real校准的重要性，但方法不够具体。GarmentLab（2025）提供了三种具体的sim2real transfer方案，在15个任务中10个成功。DiffAvatar的system identification方法也是一个参考。FitNet需要明确：(a) 使用哪种仿真器作为数据源（DiffCloth? NVIDIA Warp? 自研?），(b) 如何获取真实穿着数据进行校准（3D扫描? 压力传感器?），(c) 校准后的精度指标是什么。没有这些细节，sim-to-real就停留在口号层面。

### 5. "合身度"的定义本身就是研究问题

什么是"合身"？松量（ease）的行业标准因品类、品牌、文化、时代而不同。日本品牌的修身衬衫和美国品牌的修身衬衫有完全不同的尺寸标准。FitNet是训练一个通用的"合身度"评估器，还是为每个品牌/品类训练特定模型？通用模型需要处理这种多样性，特定模型需要大量独立的训练过程。QuantiPhy的发现（VLM "do not reason but memorize"）警告我们：neural模型可能只是记住了训练数据中的"合身"模式，而无法泛化到新的合身标准。

### 6. 输入表示的选择至关重要

FitNet接受什么作为输入？版型的2D几何？3D网格？参数化表示？人体参数？面料属性？不同的输入表示决定了模型的泛化能力。CAD-LLaMA证明hierarchical description（39.17%→80.41%）的层次化输入对性能有决定性影响。N3D-VLM的3D grounding（92.1% vs 36.3%）也表明，正确的空间表示可以带来数量级的性能提升。FitNet的输入设计值得一篇独立的探索性研究。

### 7. DeepPHY的警告：物理直觉的缺失

DeepPHY明确指出"VLMs struggle to translate descriptive physical knowledge into precise predictive control"。FitNet虽然不是VLM，但作为neural network，它同样缺乏真正的物理推理能力。面料的非线性弹性、缝线的局部刚度变化、人体运动时的动态变形——这些物理现象是否能被纯数据驱动的surrogate准确捕捉？还是说FitNet只能在"静态站立"pose上给出合理预测，而在"弯腰"、"举手"等动态pose上失效？

## 改进建议

1. **明确使用场景**：FitNet的核心价值在大规模优化（如evolutionary search中的fitness评估），而非单次设计评估——应聚焦此场景
2. **hybrid评估策略**：FitNet做快速初筛（毫秒级），DiffCloth对top-k候选做精确验证（分钟级）——两层评估
3. **uncertainty quantification**：FitNet必须输出不确定性估计，当输入接近训练分布边界时自动fallback到物理仿真
4. **利用GarmentLab的sim2real方案**：具体选择其三种方案之一作为校准方法
5. **参照DiffAvatar的system identification**：用真实穿着数据反推仿真参数，缩小sim-real gap
6. **分阶段验证**：先在单一品类（如T恤）上证明surrogate精度可接受，再扩展

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 8 | 7 | Neural surrogate for physics simulation不是新概念；在服装领域的应用有新颖性但不是8 |
| Feasibility | 6 | 5 | 训练数据生成成本、sim-to-real校准挑战被低估 |
| Impact | 8 | 8 | 维持——如果做好，FitNet对整个pipeline的加速是变革性的 |
| Clarity | 7 | 6 | 输入表示、"合身度"定义、校准方案等关键设计选择未明确 |
| Evidence | 5 | 5 | 维持——需要更多跨领域引用 |

## 补充参考文献

- **GVU Theory** (2025): "Strengthen the verifier"支持FitNet定位，但Hallucination Barrier限制neural verifier
- **DiffCloth**: 85x梯度加速——可能使surrogate不必要
- **Dress Anyone**: 5-30分钟pipeline，仿真已足够快
- **DiffAvatar**: System identification，sim-to-real校准
- **GarmentLab**: 3种sim2real方案，10/15任务成功
- **QuantiPhy**: Neural模型"memorize not reason"的警告
- **DeepPHY**: 物理推理能力的缺失
- **N3D-VLM**: 空间表示对性能的决定性影响
- **CAD-LLaMA**: Hierarchical input representation
