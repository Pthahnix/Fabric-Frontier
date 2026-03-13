<!-- markdownlint-disable -->
# Rebuttal: PatternBridge Cross-CAD Translator

> Reviewer角色：跨领域审稿人（CAD interoperability + language models for engineering）
> 审阅日期：2026-03-13
> 审阅对象：ideas/13-cross-cad-pattern-translator.md

## 主要反驳意见

### 1. Textile IR（2026）与PatternBridge高度重叠——新颖性需下调

Textile IR（2026）已经定义了**formal node types和七层验证梯度（seven-layer verification ladder）**，明确声称"Not just a file format; it is a semantic graph"。这与PatternBridge提出的Unified Pattern IR在概念上高度重叠——两者都主张建立一种中间表示来统一不同的版型格式。关键区别在于Textile IR从textile manufacturing全链路出发（从纤维到成品），而PatternBridge聚焦于CAD格式互转。但核心的IR设计理念——将版型表示为语义图、定义formal node types、建立verification层——已经被Textile IR提出。PatternBridge更准确的定位是**Textile IR概念在CAD互转场景的工程实现**，这是incremental engineering contribution而非conceptual novelty。新颖性N=6仍然偏高，建议调整为N=4-5。

### 2. LLM直接生成CAD代码可能是更简单的路径

Text-to-CadQuery已经证明LLM可以直接使用CAD API生成可执行代码。在这个范式下，**也许根本不需要IR**——让LLM直接读取DXF文件并生成CLO3D兼容代码（或反向），可能比设计和维护一个中间表示更简单、更灵活。IR的优势是formalism和可验证性，但代价是维护成本高、每增加一个新格式就需要扩展IR schema。如果LLM的code generation能力继续提升，IR方案可能变成over-engineering。

### 3. CAD-Assistant的tool augmentation是更灵活的替代方案

CAD-Assistant（2024）采用**tool-augmented VLLM**架构，其中constraint checker是最大的性能提升来源。这种架构的关键优势是：**添加新格式只需要编写tool的docstring和API wrapper**，无需修改核心IR schema。对比PatternBridge的方案：

- **PatternBridge**：新格式 → 扩展IR schema → 编写双向converter → 更新validation rules
- **CAD-Assistant**：新格式 → 编写tool wrapper + docstring → 完成

在CAD格式不断演进的现实中（CLO3D每年更新格式、新工具如Style3D的格式也在变化），CAD-Assistant的方案维护成本显著更低。

### 4. Round-trip validation是强项但需要明确容差标准

Round-trip validation（A → IR → B → IR → A'，验证A ≈ A'）是PatternBridge设计中真正有价值的贡献——但"≈"的具体含义没有定义。需要明确：

- **几何容差**：control point位置误差多少算acceptable？（0.1mm？1mm？）
- **拓扑容差**：缝合关系的保持是否允许degradation？
- **语义容差**：grain line方向、ease标注等metadata的保持标准是什么？

Textile IR的七层验证梯度可以为这些容差标准的定义提供参考框架。

### 5. 影响力评估偏高——格式转换不改变AI能力

Impact I=9对于一个格式转换工具来说偏高。CAD格式互转虽然有实际工业价值（减少手工重建版型的时间），但它不改变AI方法本身的能力——不会让pattern generation更好、不会让drape simulation更准确。它的价值是**降低工作流程摩擦**，这是engineering impact而非scientific impact。在academic publication的语境中，这更适合作为tool/system paper而非research contribution。

### 6. 版型格式的真正难点不在几何而在语义

不同CAD系统对同一版型概念的表示方式差异极大。例如"缝份"：DXF中可能是offset curve，CLO3D中是associated attribute，Gerber AccuMark中是独立的图层。**语义映射**（而非几何转换）才是真正的技术挑战。Idea对这个最难的部分讨论不足——IR如何统一这些根本不同的语义模型？

## 改进建议

1. **明确与Textile IR的关系**——是extend、implement还是compete？建议定位为Textile IR在CAD互转场景的具体实现
2. **评估LLM direct code generation作为alternative**——至少作为related work讨论，并分析在什么条件下IR方案优于direct generation
3. **借鉴CAD-Assistant的tool augmentation**——考虑hybrid架构：IR用于core geometric representation，tool augmentation处理format-specific metadata
4. **定义precise容差标准**——参考Textile IR的七层验证梯度，为round-trip validation建立quantitative acceptance criteria
5. **聚焦语义映射**作为核心技术挑战——如何处理不同CAD系统的semantic model差异
6. **降低scope**——先做2个最常见格式（DXF + CLO3D）的高质量互转，再扩展

## 评分调整建议

| 维度 | 原分 | 建议 | 理由 |
|------|------|------|------|
| Novelty (N) | 6 | 4-5 | Textile IR已提出核心概念，这是incremental engineering |
| Feasibility (F) | 7 | 7 | 确定性工程，不变 |
| Impact (I) | 9 | 6-7 | 格式转换是engineering impact，非scientific impact |
| Confidence (C) | 9 | 8 | 语义映射的难度被低估 |
| Evidence (E) | 7 | 7 | 不变 |
| Total | 38 | 32-34 | 有工业价值但学术贡献需要重新定位 |

## 补充参考文献

- **Textile IR** (2026): Formal node types, seven-layer verification ladder, semantic graph representation
- **Text-to-CadQuery**: LLM direct CAD code generation — alternative to IR approach
- **CAD-Assistant** (2024): Tool-augmented VLLM, training-free, constraint checker biggest boost
- **AIDL** (2025): DSL designed for LLMs — relevant design principles for IR
- **FALCON** (2026): Grammar-constrained output — could enforce IR schema compliance
