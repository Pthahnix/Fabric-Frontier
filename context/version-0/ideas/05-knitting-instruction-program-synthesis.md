<!-- markdownlint-disable -->
# Idea 05: 针织指令程序合成（KnitCode）

## 概述

将程序合成范式从梭织服装版型扩展到**针织指令生成**。针织与梭织是两种根本不同的服装制造方式：梭织通过裁剪面料+缝合成型，针织通过逐行逐针构建面料直接成型。当前所有AI版型生成方法都只关注梭织版型，完全忽略了针织——一个市场规模超过$1300亿的领域。本提案设计KnitCode DSL，将针织指令建模为可执行程序，实现从设计描述到机器可读针织指令的自动生成。

## 指向的Gap

- **Gap 1: DSL表达力瓶颈** — GarmentCode只覆盖梭织服装，完全不涉及针织
- **Gap 13: 非刚性面料的展平失真** — 弹性面料（针织面料为主）的版型设计需要负ease，当前方法无法处理

## 推导过程

### 从Gap到Idea的逻辑链

1. **观察**：GarmentCode及所有下游方法都假设"裁剪+缝合"的制造模式，针织完全不在其范畴内
2. **市场需求**：针织服装占全球服装市场的重要份额（T恤、毛衣、运动服、袜子等）
3. **计算针织的先行工作**：McCann et al. (SIGGRAPH 2016) 已经展示了"3D mesh → machine knitting instructions"的编译器，证明了将针织指令作为程序处理的可行性
4. **创新点**：将AI程序合成（Design2GarmentCode的方法论）与计算针织（McCann的编译器方法论）结合

## 详细设计

### 1. KnitCode DSL设计

**基本操作原语**：

| 原语 | 说明 | 参数 |
|------|------|------|
| `knit(n)` | 正针n针 | 针数 |
| `purl(n)` | 反针n针 | 针数 |
| `increase(type, n)` | 加针n针 | 加针方式（M1L/M1R/KFB/YO）, 针数 |
| `decrease(type, n)` | 减针n针 | 减针方式（K2tog/SSK/S2KP）, 针数 |
| `cable(width, direction)` | 绞花 | 宽度, 方向 |
| `bind_off(n)` | 收针 | 针数 |
| `cast_on(n)` | 起针 | 针数 |
| `repeat(pattern, times)` | 重复花样 | 花样定义, 重复次数 |
| `short_row(left, right)` | 引返 | 左/右停止位置 |

**结构化构造**：

```python
class KnitGarment:
    def __init__(self, gauge, body_measurements):
        self.gauge = gauge  # {stitches_per_cm, rows_per_cm}
        self.body = body_measurements

class Sleeve(KnitComponent):
    def construct(self):
        cast_on(self.cuff_stitches)
        ribbing(k2p2, rows=self.rib_length)
        # 袖身：逐行加针塑形
        for section in self.shaping_schedule:
            straight(section.rows)
            increase(M1L, 1, position=START)
            increase(M1R, 1, position=END)
        # 袖山：逐行减针
        for step in self.cap_shaping:
            decrease(K2tog, step.count, position=step.pos)
        bind_off(remaining)
```

### 2. 从设计到KnitCode的管道

```
设计输入（"一件圆领套头毛衣，简洁的平针编织"）
    ↓
[VLM/LLM] 提取设计参数（领型、袖型、身长、花样）
    ↓
[Gauge Calculator] 根据纱线+针号计算密度
    ↓
[Body-to-Stitch Mapper] 体型测量 → 针数/行数
    ↓
[KnitCode Generator] 生成完整KnitCode程序
    ↓
[Knitting Machine Compiler] KnitCode → 机器指令（.kc/.pat格式）
```

### 3. 关键技术挑战

**a) Gauge计算**：不同纱线+针号组合产生不同的编织密度（gauge），直接决定针数/行数的计算。需要建立纱线-针号-密度的数据库。

**b) 3D塑形指令生成**：针织的3D成型通过加针（increase）和减针（decrease）实现，其策略取决于目标3D形状。这是一个"3D surface → stitch manipulation schedule"的映射问题。

**c) 花样与塑形的协调**：复杂编织花样（绞花、蕾丝、提花）在塑形区域需要特殊处理——加减针可能打断花样的重复单元。

## 参考文献分析

### 核心参考

1. **McCann et al.** "A compiler for 3D machine knitting" (SIGGRAPH 2016)
   - 将3D mesh编译为工业针织机的指令序列
   - 证明了针织指令可以作为"程序"处理
   - 本提案的不同之处：McCann从3D mesh出发，本提案从设计描述出发

2. **Narayanan et al.** "Automatic machine knitting of 3D meshes" (SIGGRAPH 2018)
   - 扩展了McCann的工作，处理更复杂的3D形状
   - 提出了更通用的stitch scheduling算法

3. **GarmentCode** (Korosteleva & Sorkine-Hornung, SIGGRAPH 2023)
   - DSL设计理念可以迁移：参数化组件+接口+约束
   - 但GarmentCode是面向梭织的，KnitCode的原语和约束完全不同

4. **Design2GarmentCode** (Zhou et al., CVPR 2025)
   - LLM合成DSL程序的方法论可以直接复用
   - 微调LLM生成KnitCode程序

5. **Ravelry.com / FreeSewing.org**
   - 海量的社区贡献针织花样和指令
   - 可作为KnitCode的训练数据来源

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 8/10 | AI驱动的针织指令生成在学术界基本空白；将程序合成迁移到针织域是新方向 |
| **可行性** | 5/10 | 计算针织有先行工作（McCann 2016），但从设计描述到针织指令的完整管道尚未验证；针织的约束体系比梭织更复杂 |
| **影响力** | 7/10 | 针织市场巨大，但受众（手工编织者+针织工厂）与梭织版型的受众不同 |
| **清晰度** | 7/10 | DSL设计和管道结构清晰，但花样-塑形协调等技术细节需要深入研究 |
| **证据支撑** | 6/10 | McCann的编译器证明了技术路线可行，Ravelry的海量数据提供了训练素材 |
| **总分** | **33/50** | |

## 风险与挑战

1. **针织的组合爆炸**：针织花样种类繁多（数百种基本针法+无限组合），DSL需要足够灵活
2. **3D塑形的精确性**：加减针策略直接影响最终形状，计算误差会导致尺寸偏差
3. **机器兼容性**：不同针织机（家用 vs 工业，平机 vs 圆机）的指令格式不同
4. **数据标注**：现有针织花样多为自然语言描述（"k2, p2, repeat"），需要解析为结构化数据
5. **验证困难**：需要实际编织样品来验证生成的指令正确性
