<!-- markdownlint-disable -->
# Rebuttal: GarmentBench Unified Evaluation Benchmark

> Reviewer角色：跨领域审稿人（benchmark methodology + garment AI）
> 审阅日期：2026-03-13
> 审阅对象：ideas/09-garmentbench-unified-evaluation-benchmark.md

## 主要反驳意见

### 1. GLUE类比是一把双刃剑——benchmark质量危机的警告

该Idea以GLUE/ImageNet为成功范例，论证garment AI也需要统一benchmark。然而BetterBench（Stanford, 2024）的系统性评估揭示了一个令人不安的事实：**MMLU——最接近GLUE的后继benchmark——在46条质量标准中仅得5.5/15，是24个被评估benchmark中得分最低的**。更具体地说，24个benchmark中14个在模型区分上没有统计显著性，17个不提供replication scripts。"Not all benchmarks are of the same quality"这一结论直接质疑了"只要建了benchmark就能推动领域进步"的隐含假设。GarmentBench如果不从一开始就内建质量保证机制，极可能重蹈MMLU的覆辙——被广泛使用但实际上无法可靠区分模型能力。

### 2. 统一benchmark的系统性风险："scientific monoculture"

"Can We Trust AI Benchmarks?"（2025）提出了比质量更深层的批评：**benchmark本质上是normative instruments（规范性工具）**，它们不仅测量能力，还定义什么是"好的"。Koch警告单一benchmark会导致"task-driven scientific monoculture that privileges immediate, explicit, formal, quantitative evaluation"。在garment AI领域，这个问题尤为严重——一件"好版型"的定义本身是文化、审美、功能的交汇，不存在单一客观标准。GarmentBench的5-tier metrics体系隐含了一套价值判断（如几何精度优先于美学？功能性优先于创意性？），但Idea中完全没有讨论这些normative choices。谁来定义什么是"好版型"？中国国标、意大利haute couture、日本和服pattern——它们的评判标准根本不同。

### 3. SAIBench证明模块化优于单体——架构方向需要修正

SAIBench提出的SAIL DSL模式明确主张**解耦问题定义（problem）、模型接口（model）和评估指标（metrics）**，使之可独立组合。这与GarmentBench的"大一统"设计理念形成直接对立。SAIBench的论点是：单体benchmark在新任务、新指标出现时难以扩展，而模块化框架可以在不修改core的情况下添加新评估维度。对于garment AI这样快速演进且子任务高度异质的领域（2D pattern generation、3D drape simulation、grading、nesting完全不同），模块化方案显然更合理。

### 4. 评估维度远不够——SVGenius的启示

SVGenius（2025）仅针对SVG生成这一个子任务就设计了**18种评估指标**。版型生成涉及几何精度、缝份合理性、片型数量、放码一致性、面料利用率、drape效果、body fit、可制造性等多个维度，每个维度可能需要3-5个具体指标。提议的5-tier metrics体系（geometric/structural/functional/aesthetic/manufacturing）虽然方向正确，但每个tier内部的指标设计才是真正的挑战。Idea对此没有给出具体方案。

### 5. 测试集规模vs质量的权衡

500+ test cases看似可观，但BenchFrame的研究发现HumanEval到HumanEvalNext的pass@1下降了31%——**不是因为模型变差，而是因为测试用例质量和难度提高了**。这说明盲目追求数量不如精心设计difficulty stratification。GarmentBench应该包含explicit difficulty levels（如简单直线裙 → 复杂立体剪裁），并在每个difficulty level上确保statistical power。

### 6. 新颖性评估偏高

新颖性N=7可能过高。Benchmark construction methodology本身是ML领域的成熟方法论，价值主要在于执行质量而非方法创新。将已有的benchmark设计原则（如BetterBench的46条标准、SAIBench的模块化）应用到新领域，更准确的新颖性应为N=5-6。

### 7. 缺乏维护和演进计划

"Can We Trust"明确指出benchmark需要"living document"式的维护。GarmentBench没有讨论版本更新策略、数据contamination防护、以及随着领域进步如何提升benchmark难度。

## 改进建议

1. **采用SAIBench的SAIL模式**设计可组合评估框架，而非单体benchmark。定义标准化的task description language，使社区可以贡献新任务/指标/数据集
2. **应用BetterBench的46条标准进行自检**，在发布前确保质量。特别关注statistical significance、replication scripts、documentation completeness
3. **引入explicit difficulty stratification**——从简单（A-line skirt）到困难（multi-layer drape dress），确保每个level至少50个test cases以保证statistical power
4. **建立benchmark governance**——明确谁有权修改评估标准、如何处理normative disagreements（如不同文化对"好版型"的不同定义）
5. **设计anti-contamination机制**——held-out test set + periodic refresh
6. **发布maintenance roadmap**——versioning strategy、update frequency、deprecation policy

## 评分调整建议

| 维度 | 原分 | 建议 | 理由 |
|------|------|------|------|
| Novelty (N) | 7 | 5-6 | Benchmark方法论本身不新颖，应用到新领域是incremental |
| Feasibility (F) | 8 | 7 | 模块化架构比单体更complex但更sustainable |
| Impact (I) | 10 | 9 | 需要讨论monoculture风险和normative choices |
| Confidence (C) | 9 | 8 | 缺少maintenance plan降低长期confidence |
| Total | 42 | 37-38 | 仍是高分idea，但需要更成熟的benchmark methodology |

## 补充参考文献

- **BetterBench** (Stanford, 2024): 46-criteria benchmark quality assessment. MMLU scored 5.5/15
- **Can We Trust AI Benchmarks?** (2025): Benchmarks as normative instruments. Scientific monoculture warning
- **SAIBench**: SAIL DSL for modular benchmark composition. Decouples problems/models/metrics
- **SVGenius** (2025): 18 metrics for SVG evaluation — demonstrates metric complexity even for single tasks
- **BenchFrame**: HumanEval→HumanEvalNext pass@1 drops 31% — test quality > quantity
