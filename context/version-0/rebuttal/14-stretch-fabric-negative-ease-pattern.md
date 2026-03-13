<!-- markdownlint-disable -->
# Rebuttal: Stretch Fabric Negative Ease Pattern

> Reviewer角色：跨领域审稿人（computational mechanics + 医疗纺织品工程 + wearable design）
> 审阅日期：2026-03-13
> 审阅对象：ideas/14-stretch-fabric-negative-ease-pattern

## 主要反驳意见

### 1. DiffCloth已实现可微弹性布料仿真——该Idea的仿真管线不应从头构建

DiffCloth（ICLR 2023）已经实现了包含材料属性的differentiable cloth simulation，其"hat controller"实验展示了closed-loop优化与布料变形的结合，达到了**85x speedup over naive RL methods**。这意味着弹性面料仿真的differentiable infrastructure已经存在。该Idea提出的"differentiable elastic simulation optimization"如果不build on DiffCloth，就是在重复造轮子。更关键的是，DiffCloth的梯度信息可以直接用于从target body shape反向推导pattern——这正是negative ease pattern generation所需的inverse problem。Idea应明确将DiffCloth作为核心仿真引擎，专注于**negative ease特有的优化目标设计**而非仿真层实现。

### 2. 非线性弹性模型σ(ε) = E·ε + η·ε²严重过于简化

该Idea使用的二阶非线性应力-应变模型仅有2个参数（E和η），但真实针织面料的力学行为远比这复杂。Kawabata Evaluation System（KES）测量**16个独立力学参数**，涵盖拉伸、剪切、弯曲、压缩、表面摩擦和厚度。具体而言：

- **迟滞（hysteresis）**：加载和卸载曲线不重合，意味着穿上和穿着过程中面料的力学响应不同
- **粘弹性（viscoelasticity）**：应力依赖于应变速率，快速穿上和慢速穿着时面料行为不同
- **各向异性（anisotropy）**：经向和纬向弹性系数可差2-5倍，斜向（bias）弹性最大——单标量模型无法捕获方向依赖性
- **应力松弛（stress relaxation）**：持续拉伸下弹性力逐渐降低，初始回缩量≠长期穿着回缩量

将这些简化为2参数模型会导致pattern计算结果与实际穿着效果之间产生系统性偏差。

### 3. Inverse Garment已部分实现了body shape适应性版型调整

该Idea声称"不存在针对弹性面料的自动版型调整系统"，但Inverse Garment（Section 3.3）已经实现了**automated pattern grading with body shape adaptation**——根据不同体型自动调整版型参数。虽然Inverse Garment主要处理非弹性面料，但其从3D target shape到2D pattern的反问题求解框架是直接可扩展到弹性场景的。该Idea的新颖性声明需要更精确地限定：不是"自动版型调整"本身是新的，而是"在大变形弹性约束下的版型调整"是新的。这一区分决定了真正的技术增量在哪里。

### 4. 医疗压力服装工程已有严格的负ease设计方法论——不应忽视

医疗压力袜/袖（compression garments）的设计已有成熟的工程方法论，受ISO 13934和RAL-GZ 387/1等标准规范。这些标准精确定义了：

- **分级压力梯度**：踝部30-40mmHg递减至膝部15-20mmHg
- **面料选择矩阵**：弹性模量 × 纱线密度 × 编织结构的组合对压力输出的映射
- **负ease计算公式**：基于Laplace定律（P = T/R，压力=张力/曲率半径）从目标压力反算pattern缩减量

该Idea完全没有引用这一成熟领域的知识。医疗压力服装与时装负ease的区别仅在于约束的精度要求和舒适度trade-off——底层物理模型完全相同。Idea应build on这些已验证的工程方法，而非从scratch建立弹性模型。

### 5. 100,000 FEM训练样本的计算可行性存疑

该Idea提出生成100,000个FEM仿真样本用于训练surrogate model。但弹性面料在30-50%应变下的大变形FEM仿真存在严重的数值稳定性问题——单元反转（element inversion）、接触检测失败、时间步长过小等。工业级弹性仿真单个case可能需要数小时到数天。即使使用DiffCloth加速，100,000个大变形弹性仿真的总计算量仍然极其可观。Idea需要讨论：(a) physics-informed neural networks（PINN）作为替代——用物理损失函数减少所需训练样本量；(b) adaptive sampling策略——优先仿真高信息量的parameter regions；(c) multi-fidelity方法——用低精度仿真预筛选，仅对有前途的候选做高精度仿真。

## 改进建议

1. **以DiffCloth为仿真基础**：不从头构建弹性仿真管线，而是在DiffCloth的differentiable framework上设计negative ease特有的优化目标
2. **升级材料模型至各向异性非线性**：至少使用tensor-based model区分经纬方向弹性，引入KES参数体系
3. **系统性引用医疗压力服装文献**：将ISO 13934标准、Laplace定律压力计算作为技术基础，明确时装负ease与医疗压力服装的异同
4. **用PINN替代暴力FEM数据集**：physics-informed loss可将所需训练样本量减少1-2个数量级
5. **明确与Inverse Garment的技术差异**：强调大变形弹性约束是真正的增量贡献，而非"自动版型调整"这一已存在的能力
6. **加入应力松弛的时间维度**：negative ease pattern不仅需要在穿着瞬间合体，还需要在8-12小时穿着后保持合体

## 评分调整建议

| 维度 | 原分 | 建议分 | 理由 |
|------|------|--------|------|
| Novelty | 7 | 7 | 方向正确，但需更精确定位技术增量（大变形弹性约束而非一般版型调整） |
| Feasibility | 6 | 5 | 100K FEM样本计算量存疑，2参数模型过于简化，需重新评估技术路径 |
| Impact | 8 | 8 | 弹性面料市场确实巨大，维持 |
| Clarity | 8 | 7 | 缺少对医疗压力服装领域的讨论，材料模型过于简化未被说明 |
| Evidence | 6 | 5 | 未引用DiffCloth、Inverse Garment、ISO 13934等关键先行工作 |

## 补充参考文献

- **DiffCloth** (ICLR 2023): Differentiable cloth simulation with material properties, 85x speedup — 弹性面料仿真的核心基础设施
- **Inverse Garment** (Section 3.3): Automated pattern grading with body shape adaptation — 已部分实现目标功能
- **Kawabata Evaluation System (KES)**: 16参数面料力学表征体系 — 2参数模型的局限性参照
- **ISO 13934 / RAL-GZ 387/1**: 医疗压力服装标准 — 负ease设计的成熟工程方法论
- **Laplace定律 (P=T/R)**: 从目标压力反算pattern缩减量的经典物理模型
- **Physics-Informed Neural Networks (PINN)**: 用物理损失减少FEM训练样本需求 — 计算可行性解决方案
