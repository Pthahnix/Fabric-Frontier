<!-- markdownlint-disable -->
# Rebuttal: KnitCode - Knitting Instruction Program Synthesis

> Reviewer角色：跨领域审稿人（程序合成 + 纺织工程）
> 审阅日期：2026-03-13
> 审阅对象：ideas/05-knitting-instruction-program-synthesis

## 主要反驳意见

### 1. 领域关联度有限——资源分配的优先级问题

KnitCode本质上是一个独立子领域的工作。针织（knitting）与梭织（woven）服装在制造工艺、设计逻辑、CAD工具链上几乎完全不同。如果研究资源有限（通常都是），将精力投入针织指令合成而非梭织版型生成，可能是战略性失误。梭织服装占全球服装产量的主体，且与Idea 01-04、06-08形成技术协同——而KnitCode几乎无法为其他Idea提供复用价值。Total=33是8个Idea中最低的，这一排名可能准确反映了其相对优先级。

### 2. McCann et al.的针织编译器（2016）已有相当成熟的基础

Carnegie Mellon的McCann团队早在2016年就发表了从高层针织描述到机器指令的编译器工作。后续工作包括自动化针织图案设计、短行编织（short row）生成等。Idea中虽然提到了这些前驱工作，但未深入分析GarmentCode式的程序合成方法相对于编译器方法的技术优势。编译器方法已经能处理"高层描述→机器指令"的转换——KnitCode的增量贡献在哪里？是更强的自然语言理解？更灵活的图案组合？还是处理更复杂的结构（如多色提花、立体编织）？这些都需要明确。

### 3. AIDL的"为LLM设计DSL"理念同样适用——但约束更严格

AIDL提出为语言模型量身定制DSL，而非让模型适应已有DSL。这一理念同样适用于KnitCode。但针织指令有一个梭织版型不具备的特殊挑战：**每一针都有结构意义**。在梭织版型中，略微偏移一条曲线可能只影响合身度；在针织中，漏掉一针可能导致整件织物散架。这种"零容错"特性使得constraint solver的角色比梭织场景更为关键。FALCON的grammar-constrained decoding在这里尤为相关——针织指令可以被精确定义为一种formal grammar，使得100%结构正确的生成成为可能。

### 4. Text-to-CadQuery的教训：基于Python的API可能比全新DSL更实际

Text-to-CadQuery（2025）的一个关键发现是：一个124M参数的模型通过直接生成Python API调用，就能超越363M的专用模型。这暗示了一条更务实的路径——与其为针织设计全新的DSL（需要生态系统从零构建），不如基于Python封装现有的针织操作库，让LLM生成Python代码来控制针织过程。Python生态成熟、工具丰富、社区庞大，这些优势在DSL设计中不可忽视。

### 5. 物理验证更加困难

针织物的物理仿真比梭织复杂得多。纱线是一维弹性体，编织结构产生高度非线性的力学响应，松紧度（gauge）对最终尺寸有巨大影响。DiffCloth主要针对梭织物设计（三角网格），能否有效模拟针织物的弹性行为？如果物理仿真不可靠，GVU理论中的external verifier就无法实现——自改进循环的基础缺失。Idea需要讨论针织物仿真的现状和局限。

### 6. 数据集的获取更加困难

梭织版型至少有FreeSewing等开源来源。针织领域的数字化程度更低——大量图案仍以纸质图表或口头传授的形式存在。Ravelry社区虽然庞大，但其图案多为非结构化的自然语言描述（"起40针，织两行下针，两行上针..."），将其转化为结构化DSL程序本身就是一个研究挑战。Idea的可行性评分F=5可能还是过高了。

### 7. 工业针织机的指令系统差异巨大

Stoll、Shima Seiki、Steiger等主要针织机厂商各有完全不兼容的指令系统。KnitCode的DSL编译到哪个目标？如果是通用中间表示，需要为每种机器编写后端——这是一个庞大的工程任务。如果只支持一种机器，应用范围大幅缩小。McCann等人选择了Knitout作为中间表示来解决这个问题，但Knitout的采用度仍然有限。

## 改进建议

1. **明确差异化定位**：相对于McCann编译器方法，KnitCode的核心技术优势是什么？建议聚焦于自然语言→结构化指令的转换，这是编译器方法不擅长的
2. **考虑Python API路线**：参照Text-to-CadQuery，评估基于Python封装Knitout的方案是否比全新DSL更实际
3. **引入grammar-constrained generation**：FALCON的方法直接适用于针织指令——定义formal grammar，保证生成的每条指令都结构正确
4. **缩小scope**：不追求通用针织合成，先聚焦一个子领域（如2D平面编织图案），证明可行性后再扩展
5. **与其他Idea建立技术桥梁**：如果KnitCode的DSL设计方法论（而非具体DSL）能迁移到梭织版型，则提升了整体研究portfolio的一致性

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 8 | 6 | McCann编译器方法已有相当基础，增量新颖性需更清晰的论证 |
| Feasibility | 5 | 4 | 数据获取困难、物理仿真局限、机器指令系统碎片化 |
| Impact | 7 | 6 | 与梭织主线的协同有限，独立价值需要更强的论证 |
| Clarity | 7 | 7 | 维持——问题定义是清晰的 |
| Evidence | 6 | 5 | 缺少对McCann系列工作和Knitout生态的深入讨论 |

## 补充参考文献

- **McCann et al. (2016)**: 针织编译器，高层描述→机器指令
- **Knitout**: 针织中间表示标准
- **AIDL** (2025): 为LLM设计的DSL，constraint solver卸载
- **Text-to-CadQuery** (2025): Python API路线，124M超越363M
- **FALCON** (2026): Grammar-constrained decoding，100%结构正确性
- **DiffCloth**: 可微仿真——但主要针对梭织物
- **GVU Theory** (2025): External verifier的必要性
