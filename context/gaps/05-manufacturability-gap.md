<!-- markdownlint-disable -->
# 方向五：AI版型的可制造性鸿沟

## 概述

产学界公认的关键瓶颈：AI能生成"好看的版片"，但距离工厂可裁可缝仍有巨大鸿沟。研究表明，AI在缝份（seam allowance）、标记点（notch）、分码推板（grading）等环节仍需人工介入；但协作模式下可实现46%的面料节省。本方向的5个Gap共同构成了从"学术版型"到"可生产版型"的系统性障碍。

---

## Gap 17: 缝份（Seam Allowance）的自动生成

### 问题描述

所有学术版型生成方法输出的是**净版**（net pattern，不含缝份），而工厂实际裁剪需要的是**毛版**（gross pattern，含缝份）。缝份是缝合时被折入内侧的面料余量，其宽度因部位、工艺、面料类型而异。没有缝份的版型在工业生产中完全不可用。

### 缝份规则的复杂性

| 部位 | 典型缝份宽度 | 影响因素 |
|------|-------------|---------|
| 领口 | 0.7-1.0 cm | 曲率大的部位需要较窄缝份 |
| 肩缝 | 1.0 cm | 标准缝份 |
| 侧缝 | 1.0-1.5 cm | 可能需要留改大余量 |
| 下摆 | 2.0-4.0 cm | 需要折边 |
| 拉链口 | 1.5 cm | 需要容纳拉链齿 |
| 袖窿 | 1.0 cm | 曲线部位 |

### 现有方法的局限

- **GarmentCode DSL**: 完全没有缝份概念，输出的panel边界是净版边界
- **Design2GarmentCode, AIpparel, ChatGarment, GarmentDiffusion, DressCode**: 所有方法输出panel几何形状+stitch信息，从不包含缝份
- **Style3D AI商业产品**: 声称可处理缝份，但学术论文中无任何描述
- **ASTM D6673标准**: 定义了工业缝份规范，但没有任何AI系统整合此标准

### 证据来源

1. **所有核心论文的输出格式**: 均为panel顶点坐标+edge参数+stitch tags，无缝份字段
2. **fashioninsta.ai报告**: "AI generates basic patterns but needs human oversight for seam allowances"
3. **实际生产流程**: 制版师在完成净版后，必须手动或用CAD软件添加缝份，这是额外的必要步骤

### 影响程度

**高** — 没有缝份的版型无法进入裁剪车间。这是AI版型走向实际生产的第一道硬性障碍。

---

## Gap 18: 标记点（Notch）和对位点的缺失

### 问题描述

工业版型上必须包含大量标记信息，用于指导裁剪和缝合。这些标记包括：

1. **Notch（剪口/三角缺口）**: 版片边缘的V形或T形标记，用于缝合时对位。例如袖片上的notch对准身片的肩点。
2. **Grain Line（布纹线）**: 标示面料的经纱方向，面料必须按此方向裁剪以确保垂感和拉伸方向正确。
3. **Dart Points（省尖点）**: 标记省道的精确终止位置。
4. **Pleat Marks（褶裥标记）**: 标示褶裥的折叠方向和位置。
5. **Button/Buttonhole Positions**: 纽扣和扣眼的精确位置。

### 现有方法的局限

- **所有论文（Design2GarmentCode, AIpparel, ChatGarment, GarmentDiffusion, DressCode, SewFormer）**: 输出仅包含panel几何形状和stitch信息（哪些边缝合在一起），完全不包含notch、grain line、dart point等标记
- **GarmentCode DSL**: 定义panel形状和参数化构件，但不输出裁剪标记
- **Stitch tags**: 现有方法用per-edge stitch tags（3D坐标中点）来编码缝合关系，但这不等于notch——notch需要精确的沿边位置标记

### 证据来源

1. **DressCode论文** Section 3.1: "Each stitch tag $S_{i,j} \in \mathbb{R}^3$ is based on the 3D placement of the corresponding edge" — stitch tags编码缝合配对关系，但不提供缝合对位的精确标记
2. **GarmentDiffusion论文**: Edge representation包含start point、control points、arc parameters、stitch tags、stitch flags — 没有notch信息
3. **工业实践**: 一件普通衬衫的版型上通常有20-30个notch，一条裤子有15-20个notch。没有这些标记，缝纫工人无法正确对位缝合

### 影响程度

**极高** — 一个没有notch和grain line的版型在工厂中等于"废纸"。这是可制造性的最基本要求。

---

## Gap 19: 面料约束的缺失

### 问题描述

AI生成版型时完全不考虑面料的物理属性和使用约束。然而在实际制版中，面料特性直接决定版型设计的多个关键参数。

### 被忽视的面料约束

| 约束类型 | 说明 | 影响 |
|----------|------|------|
| **幅宽** | 面料有固定宽度（110/140/150cm），单片版型宽度不能超过幅宽 | 影响版型分片策略 |
| **方向性** | 经向（warp）和纬向（weft）拉伸性不同，某些面料有倒毛方向 | 影响grain line放置和裁片方向 |
| **弹性** | 不同方向弹性不同，影响ease量 | 弹性面料需要负ease |
| **图案对花** | 条纹/格纹面料需要跨版片图案连续 | 极难的约束满足问题 |
| **面料损耗率** | 不同面料类型有不同裁剪损耗率 | 影响排料和用料计算 |

### 现有方法的局限

- **AIpparel** limitations: "Fabricating the generated garments is another interesting direction, which requires consideration of physical and material constraints during sewing pattern prediction"
- **SewingLDM** (ICCV 2025): 唯一将body shape作为条件的方法，但仍完全忽略面料属性
- **DiffAvatar**: 恢复材料参数（弹性模量、弯曲刚度），但这些参数不反馈到版型设计
- **所有方法**: 没有面料幅宽约束（生成的版片可能超过面料宽度）

### 证据来源

1. **AIpparel论文** Discussion section: 明确将面料约束列为future work
2. **实际制版**: 使用丝绸vs牛仔布做同一款衬衫，版型的ease量、省道设计、缝份宽度都不同
3. **图案对花**: 使用格纹面料时，版片在面料上的放置位置直接受到图案重复单元约束

### 影响程度

**高** — 面料是服装的"材料"，完全不考虑材料属性的设计等于"纸上谈兵"。

---

## Gap 20: 排料优化（Marker Making）

### 问题描述

排料（Marker Making / Nesting）是将所有版片在面料上最优排列以最小化面料浪费的过程。这是一个NP-hard的2D装箱问题。当前AI版型生成与排料优化完全脱节——版型形状和排料效率应该被联合优化。

### 现有状况

- **AI版型生成**: 完全不考虑生成的版片形状是否有利于排料
- **工业排料软件**: Gerber AccuMark Nest、Lectra Diamino使用启发式算法（模拟退火、遗传算法），不与版型设计耦合
- **联合优化**: 没有任何方法同时优化版型形状和排料效率
- **面料利用率**: 行业平均面料利用率约75-85%，意味着15-25%的面料被浪费

### 证据来源

1. **fashioninsta.ai**: "46% fabric reduction possible" — 说明AI协作有巨大节省空间
2. **排料问题的计算复杂度**: 不规则形状的2D nesting是NP-hard（Wascher et al., 2007）
3. **版型形状影响排料**: 版片的外轮廓曲率、凹凸程度直接影响排料效率。如果版型生成时考虑"排料友好性"，可能在不影响穿着效果的前提下大幅提升面料利用率

### 影响程度

**极高** — 面料成本通常占服装制造成本的60-70%。提升面料利用率1%就意味着巨大的经济效益。联合优化版型形状与排料效率具有巨大的工业价值和学术价值。

---

## Gap 21: 工业格式输出

### 问题描述

没有任何学术方法直接输出DXF（Drawing Exchange Format）、ASTM D6673、AAMA等工业标准格式。学术输出（SVG、JSON、程序参数、点坐标）与工业CAD系统的输入之间存在巨大鸿沟。

### 学术输出 vs 工业需求

| 学术输出 | 工业需要 |
|----------|----------|
| Panel顶点坐标列表 | DXF格式文件（含图层、线型、标注） |
| Stitch tags（3D坐标） | ASTM格式缝合定义 |
| JSON参数文件 | CAD文件（可导入CLO3D/Optitex/Lectra） |
| GarmentCode程序 | 工业级参数化模板 |
| 无缝份/无标记 | 完整毛版（含缝份、notch、grain line、标注） |

### 现有方法的局限

- **Design2GarmentCode**: 展示了与Style3D Studio的集成（用于可视化），但不输出工业格式
- **Textile IR** (Teikari & Fuenmayor, 2026, arXiv:2601.02792): 提出双向中间表征概念，旨在统一学术和工业格式，但仅为position paper，无实现
- **Style3D AI商业产品**: 声称支持DXF输出，但这是商业实现，学术研究中完全空白
- **Valentina/Seamly2D**: 开源制版CAD软件，使用完全不同于GarmentCode的参数化系统

### 证据来源

1. **Design2GarmentCode论文**: References 列出了Assyst、Modaris、Optitex、Valentina等工业工具但不提供互操作
2. **DXF格式规范**: 需要包含entities（LINE, ARC, POLYLINE）、layers（PATTERN, SEAM_ALLOWANCE, NOTCH, GRAIN_LINE）等结构化信息
3. **ASTM D6673**: 定义了数字版型数据交换的标准格式，包括panel geometry、seam connections、grading data等

### 影响程度

**极高** — 这是学术研究落地的"最后一公里"。不能输出工业格式，所有学术成果都只能停留在实验室中。
