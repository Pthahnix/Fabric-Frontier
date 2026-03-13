<!-- markdownlint-disable -->
# Idea 13 v1: 混合架构跨CAD版型翻译器（PatternBridge-H）

## 概述

构建一个**混合架构（Hybrid Architecture）**的跨CAD版型翻译系统——PatternBridge-H，将传统IR方案与LLM tool-augmented方案相结合。核心设计分为两层：（1）**Geometric IR层**——处理确定性的几何拓扑转换，保障round-trip精度；（2）**Semantic Agent层**——利用tool-augmented VLLM处理不同CAD系统间的语义映射（seam allowance、grain line、notch等元数据），解决格式特定的语义差异。这种混合架构在保持IR的formalism与可验证性的同时，通过LLM的灵活性降低新格式接入的维护成本。

## 指向的Gap

- **Gap 4: 跨CAD互操作性** — 不同学术方法和工业工具使用不兼容的版型格式
- **Gap 21: 工业格式输出** — 没有任何学术方法直接输出DXF、ASTM D6673等工业格式

## 推导过程

1. **Textile IR (Teikari & Fuenmayor, 2026)的概念基础**：该position paper提出了包含七层验证梯度（Seven-Layer Verification Ladder）的语义图中间表征，定义了formal node types（PatternPiece, SeamEdge, Dart, Notch, MaterialRegion），为版型IR设计提供了理论框架。PatternBridge-H明确定位为Textile IR在CAD互转场景的**工程实现与扩展**
2. **CAD-Assistant (Mallis et al., 2024)的架构启发**：tool-augmented VLLM在通用CAD任务中表现优异，其核心优势在于**添加新格式只需编写tool的docstring和API wrapper**，无需修改核心schema。PatternBridge-H借鉴这一思路处理format-specific的语义元数据
3. **Text-to-CadQuery (2025)与STEP-LLM (2026)的范式验证**：LLM直接生成CAD代码（CadQuery、STEP格式）的可行性已被验证——Text-to-CadQuery在170K样本上实现69.3% exact match，STEP-LLM通过DFS reserialization + RL将Chamfer Distance降至0.53。但这些工作针对的是3D机械零件，**garment pattern的2D语义（seam allowance、dart、grain line）远比3D solid简单**，因此LLM辅助语义映射在服装版型场景更为可行
4. **工程价值不变**：即使AI版型生成方法完美，如果输出不能导入到Gerber/Lectra/CLO3D中，工厂和设计师就无法使用
5. **Apparel CAD Pattern Data Interchange (基于XML)**：早期工业研究已探索基于XML的服装CAD版型数据交换格式，验证了中间格式方案在服装领域的需求，但缺乏现代AI辅助的语义理解能力

## 详细设计

### 1. 混合架构总览

```
输入格式 → [Format Parser] → [Geometric IR] ←→ [Semantic Agent] → [Format Generator] → 输出格式
                                    ↑                    ↑
                              确定性几何转换        LLM辅助语义映射
                              （可验证，精确）     （灵活，可扩展）
```

**设计原则**：几何部分用IR保障精度，语义部分用LLM保障灵活性。

### 2. Geometric IR层（确定性）

基于Textile IR的node types设计，聚焦CAD互转所需的核心几何信息：

```json
{
  "version": "1.0",
  "metadata": {
    "garment_type": "women_blouse",
    "size_system": "EU",
    "base_size": 38
  },
  "panels": [
    {
      "id": "front_bodice",
      "geometry": {
        "type": "parametric_curves",
        "curves": [
          {"type": "line", "start": [0, 0], "end": [20, 0]},
          {"type": "bezier", "control_points": [[20,0], [22,5], [21,15], [18,20]]},
          {"type": "arc", "center": [10, 20], "radius": 8, "start_angle": 0, "end_angle": 90}
        ]
      },
      "topology": {
        "closed": true,
        "winding": "CCW",
        "edge_adjacency": [[0,1], [1,2], [2,0]]
      }
    }
  ],
  "stitches": [
    {"panel_a": "front_bodice", "edge_a": 1,
     "panel_b": "back_bodice", "edge_b": 0,
     "seam_type": "plain_seam"}
  ]
}
```

Geometric IR**只包含**曲线几何、拓扑关系和缝合连接——这些信息在所有CAD系统中的表示方式本质相同，转换是确定性的。

### 3. Semantic Agent层（LLM辅助）

处理不同CAD系统间语义模型差异的核心挑战：

| 语义概念 | DXF表示 | CLO3D表示 | Gerber AccuMark表示 | 语义映射策略 |
|----------|---------|-----------|---------------------|-------------|
| 缝份(seam allowance) | offset curve (独立几何) | associated attribute (元数据) | 独立图层 | LLM解析源格式语义 → 统一表示 → 目标格式生成 |
| 布纹线(grain line) | LINE entity on GRAIN层 | JSON attribute | 标记线实体 | 格式特定tool wrapper |
| 剪口(notch) | 短线段或V形 | notch object | 图层标记 | 模式识别 + 语义推断 |
| 省道(dart) | 几何折叠线 | dart object with fold angle | 配对标记点 | 拓扑分析 + 语义重建 |

**Tool-augmented架构**（借鉴CAD-Assistant）：
```
Semantic Agent (VLLM Planner)
  ├── Tool: DXF Layer Analyzer     # 解析DXF图层语义
  ├── Tool: CLO3D Attribute Parser # 解析CLO3D属性元数据
  ├── Tool: Gerber Format Reader   # 解析AccuMark格式
  ├── Tool: ASTM D6673 Validator   # 验证ASTM标准合规性
  ├── Tool: Semantic Mapper        # 跨格式语义对齐
  └── Tool: Constraint Checker     # 验证语义完整性（借鉴CAD-Assistant最大性能提升来源）
```

**新格式接入成本对比**：

| 方案 | 新格式接入步骤 | 维护成本 |
|------|---------------|---------|
| 纯IR方案 (v0) | 扩展IR schema → 编写双向converter → 更新validation rules | 高 |
| 纯LLM方案 | 编写prompt + few-shot examples | 低但不可靠 |
| **混合方案 (v1)** | 几何：复用IR converter；语义：编写tool wrapper + docstring | 中，可控 |

### 4. 验证机制（借鉴Textile IR七层验证梯度）

**分层验证（Hierarchical Verification）**：

| 层级 | 验证内容 | 成本 | 容差标准 |
|------|---------|------|---------|
| L1: 语法验证 | IR JSON schema合法性 | 即时 | 严格：必须通过 |
| L2: 拓扑验证 | Panel闭合性、边邻接一致性 | 毫秒级 | 严格：winding方向一致 |
| L3: 几何验证 | 缝合边长度匹配 | 毫秒级 | 边长偏差 < 0.5mm |
| L4: 语义验证 | 缝份/布纹线/剪口完整性 | 秒级 | 信息无丢失（可有格式适配） |
| L5: Round-trip验证 | A → IR → B → IR → A'，验证A ≈ A' | 秒级 | 见下方精确容差 |

**Round-trip精确容差标准**：
- **几何容差**：control point位置误差 < 0.1mm；Panel面积偏差 < 0.1%
- **拓扑容差**：缝合关系必须完全保持（zero degradation），缝合边长度匹配误差 < 0.5mm
- **语义容差**：grain line方向偏差 < 1°；seam allowance宽度偏差 < 0.1mm；ease标注保持原始数值
- **曲线精度**：Hausdorff distance < 0.5mm（较v0的1mm更严格，反映工业实际需求）

### 5. 实现路径

**Phase 1：DXF ↔ CLO3D高质量互转**（scope缩减，聚焦最高价值格式对）
- 使用Python实现核心Geometric IR转换器
- DXF读写：ezdxf库
- CLO3D：逆向分析.zprj格式（ZIP包含JSON+binary mesh）
- Semantic Agent：基于GPT-4o/Claude的tool-augmented架构

**Phase 2：扩展ASTM D6673 + SVG**
- ASTM D6673标准化XML/ZIP包
- SVG带图层的通用预览格式

**Phase 3：学术格式接入**
- GarmentCode程序 → 执行 → IR
- SewingGPT tokens → 解码 → IR
- ChatGarment JSON → 直接解析 → IR

### 6. 与LLM Direct Generation的关系

v0未讨论LLM直接生成CAD代码作为替代方案。基于新研究的分析：

| 维度 | IR混合方案 (PatternBridge-H) | LLM Direct Generation |
|------|---------------------------|----------------------|
| 精度保障 | Geometric IR提供确定性转换，可数学证明 | 依赖LLM生成质量，有hallucination风险 |
| 可验证性 | 分层验证，每层有明确容差 | 只能通过结果验证（end-to-end） |
| 新格式扩展 | 语义层tool wrapper即可 | 需要新的training data或few-shot examples |
| 适用场景 | 高精度工业场景（工厂打版） | 快速原型预览（设计师探索） |

**结论**：两种方案互补而非竞争。PatternBridge-H的Semantic Agent层本质上就是一种受控的LLM generation，只是限制在语义元数据层面而非几何层面。

## 参考文献分析

1. **Textile IR** (Teikari & Fuenmayor, 2026, arXiv:2601.02792): 提出七层验证梯度和语义图IR概念，定义了PatternPiece/SeamEdge/Dart/Notch/MaterialRegion等formal node types。PatternBridge-H明确定位为其CAD互转场景的工程实现
2. **CAD-Assistant** (Mallis et al., 2024): Tool-augmented VLLM框架，training-free，constraint checker是最大性能提升来源（+7.4% accuracy）。其"添加新工具只需docstring"的扩展模式直接启发了Semantic Agent层设计
3. **Text-to-CadQuery** (2025): 证明LLM可以直接生成CAD代码（CadQuery），在170K样本上达到69.3% exact match，验证了LLM在CAD代码生成的可行性
4. **STEP-LLM** (Shi et al., 2026, arXiv:2601.12641): 首个直接生成STEP文件的框架，DFS reserialization + RL alignment将MSCD降至0.53，证明LLM可以处理结构化CAD格式
5. **AIDL** (2025, arXiv:2502.09819): 为LLM设计的solver-aided CAD DSL，通过solver offloading空间推理任务。其"限制LLM做spatial reasoning，让solver做精确计算"的设计哲学与PatternBridge-H的混合架构异曲同工
6. **Design2GarmentCode** (CVPR 2025): 展示了Style3D集成但不输出工业格式
7. **DXF格式规范**: ENTITIES、LAYERS、BLOCKS等结构，公开可查
8. **ASTM D6673**: 数字版型数据交换标准格式
9. **Valentina/Seamly2D**: 开源CAD，源代码可参考格式解析
10. **Apparel CAD Pattern Data Interchange (XML-based)**: 早期工业研究探索基于XML的服装版型交换格式

## 五维评分

| 维度 | v0分数 | v1分数 | 调整理由 |
|------|--------|--------|---------|
| **新颖性** | 6/10 | 5/10 | Textile IR已提出核心IR概念和七层验证框架；PatternBridge-H的增量贡献在于：(1)混合架构将IR与tool-augmented LLM结合，(2)聚焦CAD互转的精确容差标准定义。这是engineering novelty而非conceptual novelty |
| **可行性** | 7/10 | 7/10 | 几何转换是确定性工程任务；语义映射借助LLM降低了实现难度；Phase 1聚焦DXF↔CLO3D两个格式使scope更可控 |
| **影响力** | 9/10 | 7/10 | 接受rebuttal观点：格式转换是engineering impact而非scientific impact，不改变AI方法本身的能力。但作为tool/system paper仍有较高工业应用价值 |
| **清晰度** | 9/10 | 9/10 | 混合架构设计、分层验证、精确容差标准都有明确定义 |
| **证据支撑** | 7/10 | 8/10 | 新增CAD-Assistant、Text-to-CadQuery、STEP-LLM、AIDL等多个实证参考；Textile IR的七层验证提供了理论框架；容差标准有工业惯例支撑 |
| **总分** | **38/50** | **36/50** | 新颖性和影响力下调反映更诚实的定位（engineering contribution），证据支撑上调反映更充分的文献基础 |

## 风险与挑战

1. **工业格式的封闭性**：Gerber/Lectra/CLO3D的文件格式部分是私有的，逆向分析可能不完整。Phase 1聚焦DXF（公开标准）+ CLO3D（社区已有逆向经验）降低风险
2. **语义映射的边界情况**：LLM在处理罕见的格式特定语义时可能出错。Constraint Checker工具提供安全网，但无法覆盖所有edge case
3. **曲线精度损失**：不同格式表示曲线的方式不同（Bezier vs arc vs polyline），Geometric IR层通过统一参数化曲线表示减少但无法完全消除精度损失
4. **维护成本**：虽然混合架构降低了新格式接入成本，但Geometric IR层的converter仍需工程维护
5. **LLM依赖风险**：Semantic Agent层依赖外部LLM API，引入了latency、成本和服务可用性风险

## v0→v1 变更说明

### 核心架构变更
- **v0**: 纯IR方案——所有信息（几何+语义）都通过统一的Pattern IR表征和确定性converter处理
- **v1**: 混合架构——将几何（确定性IR）与语义（tool-augmented LLM）分离处理，结合两种方案的优势

### 响应Rebuttal的具体调整

| Rebuttal意见 | 响应方式 |
|-------------|---------|
| 1. 与Textile IR重叠，新颖性偏高 | 明确定位为Textile IR的CAD互转工程实现；新颖性N从6降至5；借鉴其七层验证梯度设计分层验证 |
| 2. LLM直接生成CAD代码可能更简单 | 新增"与LLM Direct Generation的关系"专节分析；结论：两者互补，PatternBridge-H的Semantic Agent层本质是受控的LLM generation |
| 3. CAD-Assistant的tool augmentation更灵活 | 采纳：Semantic Agent层直接借鉴CAD-Assistant的tool-augmented架构，新格式只需tool wrapper + docstring |
| 4. Round-trip validation需要明确容差 | 定义了三类精确容差标准：几何容差(0.1mm)、拓扑容差(zero degradation)、语义容差(方向1°/宽度0.1mm) |
| 5. 影响力评估偏高 | 接受：Impact从9降至7，重新定位为engineering impact / tool paper |
| 6. 语义映射才是真正难点 | 将语义映射提升为核心技术挑战，设计专门的Semantic Agent层处理；详细列出四个核心语义概念在三个主要CAD系统中的表示差异 |

### 新增文献支撑
- CAD-Assistant (2024): tool-augmented VLLM架构 → 启发Semantic Agent层设计
- Text-to-CadQuery (2025): LLM生成CAD代码可行性 → 支持LLM辅助语义映射的合理性
- STEP-LLM (2026): LLM处理结构化CAD格式 → 验证LLM在CAD领域的能力边界
- AIDL (2025): solver-aided DSL设计哲学 → 支持"LLM做高层推理、solver/IR做精确计算"的分工原则

### Scope缩减
- v0试图覆盖5种工业格式 + 5种学术格式的全面互转
- v1采用三阶段渐进实现：Phase 1仅聚焦DXF↔CLO3D，验证核心架构后再扩展
