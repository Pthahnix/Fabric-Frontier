<!-- markdownlint-disable -->
# Idea 01: GarmentCode++ 自学习DSL扩展

## 概述

受通用程序合成领域DreamCoder（Ellis et al., NeurIPS 2021）的启发，提出GarmentCode++：一个能够**自动发现和积累**版型构造子程序的自学习DSL系统。与当前GarmentCode需要专家手工添加新模板不同，GarmentCode++通过"library learning"机制，在解决新设计任务的过程中自动归纳出可复用的版型构造原语，持续扩展DSL的表达能力边界。

## 指向的Gap

- **Gap 1: DSL表达力瓶颈** — GarmentCode仅覆盖有限品类（上衣/裤装/裙装/连衣裙），无法表示口袋、特殊领型、非流形结构等。扩展方式为手工添加模板，不可扩展。
- **Gap 2: 程序合成的泛化能力** — 当前方法（Design2GarmentCode, ChatGarment）本质是"模板填参"而非真正的程序合成，无法创造新的版型拓扑结构。

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察**：GarmentCode的表达力由其预定义的组件库决定。ChatGarment为了增加覆盖范围创建了GarmentCodeRC扩展版，但扩展方式是"逐个品类手工添加"。
2. **类比**：在通用程序合成领域，DreamCoder展示了一种根本不同的扩展策略——系统在解决任务的过程中自动发现可复用的子程序（lambda abstractions），并将其添加到DSL库中。经过多轮"wake-sleep"训练后，DSL的表达能力自动增长。
3. **迁移**：将DreamCoder的library learning机制迁移到版型DSL领域。版型构造中存在大量可复用的子模式（如"一字领构造"、"公主线分割"、"活褶构造"），这些模式在不同款式中反复出现，非常适合被自动归纳为库函数。
4. **创新点**：DreamCoder处理的是数学/字符串等简单域，版型构造的域特征（几何约束、缝合拓扑、物理可行性）带来了新的研究挑战。

### 相关工作对比

| 方法 | 扩展方式 | 自动化程度 | 局限 |
|------|---------|-----------|------|
| GarmentCode (SIGGRAPH 2023) | 专家手工定义 | 完全手工 | 新品类需要大量编程 |
| ChatGarment GarmentCodeRC | 专家手工扩展 | 完全手工 | 论文承认仍需"expanding garment type coverage" |
| Design2GarmentCode | 不扩展DSL | — | 限于现有模板空间 |
| DreamCoder (NeurIPS 2021) | 自动library learning | 全自动 | 仅用于简单域 |
| **GarmentCode++ (本提案)** | 自动library learning | 半自动 | 需要版型域的适配 |

## 详细设计

### 1. 系统架构

```
输入：新设计任务（图像/文本 → 目标版型）
    ↓
[神经引导搜索器] — 使用当前DSL库尝试合成程序
    ↓ 成功 → 输出程序
    ↓ 失败 →
[抽象发现器] — 分析失败原因，在已有程序中寻找可提取的子模式
    ↓
[库更新器] — 将新发现的子模式添加为DSL库函数
    ↓
[重试合成] — 使用扩展后的DSL库重新尝试合成
```

### 2. 核心组件

**a) 神经引导搜索器（Neural-Guided Search）**
- 使用预训练LLM（如CodeLlama）作为程序合成的引导模型
- 输入：设计描述 + 当前DSL库的文档
- 输出：候选GarmentCode程序
- 关键：当LLM无法用现有DSL表达目标设计时，标记为"表达力不足"

**b) 抽象发现器（Abstraction Discovery）**
- 输入：一批成功合成的程序
- 方法：anti-unification（反合一）——找到多个程序中共同的结构模式，将差异部分参数化
- 输出：新的参数化子程序（如 `collar(type, width, depth)` 从多个具体领子构造中归纳）
- 版型域适配：子程序必须满足几何闭合约束（panel必须是闭合多边形）和缝合一致性约束

**c) 库更新器（Library Update）**
- 验证新子程序的有效性：
  - 几何验证：生成的panel是否合法（闭合、无自交）
  - 物理验证：仿真后是否可穿着（无穿透、合理悬垂）
  - 复用验证：新子程序是否在多个设计中被使用
- 只有通过所有验证的子程序才被添加到库中

### 3. 训练流程（Wake-Sleep循环）

**Wake阶段**：
- 给定设计任务集合，使用当前DSL+搜索器尝试合成所有任务的程序
- 收集成功和失败的案例

**Sleep阶段**：
- 对成功案例进行抽象发现，提取可复用子程序
- 更新DSL库
- 使用扩展后的DSL重新压缩（re-express）已有程序，验证新原语的有效性

**迭代**：重复Wake-Sleep，DSL库逐步增长

### 4. 数据需求

- 初始DSL：现有GarmentCode及其组件库
- 任务集：需要大量**超出当前DSL表达力**的设计任务（从时尚杂志、电商图片、设计师手稿中收集）
- 验证数据：每个新子程序需要多个使用实例来验证其复用性

## 参考文献分析

### 核心参考

1. **DreamCoder** (Ellis et al., NeurIPS 2021)
   - 在LIST、REGEX、LOGO等域上展示了library learning的有效性
   - Wake-Sleep训练框架可直接借鉴
   - 局限：处理的域远比版型构造简单，版型域的几何和物理约束是新挑战

2. **GarmentCode** (Korosteleva & Sorkine-Hornung, SIGGRAPH 2023)
   - 提供了初始DSL基础
   - 其参数化设计（组件+接口）天然适合子程序抽象
   - 关键信息：DSL使用Python实现，组件以类的形式组织

3. **Design2GarmentCode** (Zhou et al., CVPR 2025)
   - 展示了LLM合成GarmentCode程序的可行性
   - 论文承认"cannot substantially alter GarmentCode's underlying structure"——正是本提案要解决的问题
   - 其LLM微调方法可作为神经引导搜索器的基础

4. **ChatGarment** (Bian et al., CVPR 2025)
   - GarmentCodeRC的创建说明DSL确实需要扩展
   - 多轮对话编辑框架可与library learning结合——用户反馈帮助发现新的构造需求

5. **AIpparel** (Nakayama et al., CVPR 2025)
   - "constrained to garments representable by manifold surfaces"——DSL表达力限制的直接证据
   - 其tokenization方法可作为另一种版型表示的参考

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 9/10 | DreamCoder的library learning从未被应用于版型DSL领域，是全新的研究方向 |
| **可行性** | 5/10 | DreamCoder在简单域上有效，但版型域的几何/物理约束大幅增加难度；需要大量超出现有DSL的设计任务数据 |
| **影响力** | 9/10 | 如果成功，将根本解决DSL表达力瓶颈，使版型生成的设计空间从有限到理论上无限可扩展 |
| **清晰度** | 7/10 | 整体框架清晰，但抽象发现器在版型域的具体实现细节需要进一步研究 |
| **证据支撑** | 6/10 | DreamCoder证明了library learning的通用性，GarmentCode的组件化设计暗示了子程序可提取性，但缺乏版型域的直接实验证据 |
| **总分** | **36/50** | |

## 风险与挑战

1. **域复杂度鸿沟**：DreamCoder处理的域（列表操作、正则表达式）与版型构造的复杂度差了数个数量级。版型的几何闭合、缝合拓扑、物理可行性约束可能使搜索空间过大
2. **任务集构建**：需要大量"刚好超出当前DSL"的设计任务，过简单的任务无法推动DSL扩展，过复杂的任务可能无法合成
3. **质量控制**：自动发现的子程序可能生成几何合法但制版上不合理的版型（如不符合工艺规范的拼缝位置）
4. **评估困难**：如何衡量DSL的"表达力增长"？需要定义版型设计空间的覆盖率指标
