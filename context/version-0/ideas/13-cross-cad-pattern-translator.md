<!-- markdownlint-disable -->
# Idea 13: 跨CAD版型翻译器（PatternBridge）

## 概述

构建一个通用的版型格式翻译器——PatternBridge，通过设计一个包含所有必要信息的中间表征（Intermediate Representation, IR），实现学术版型表示与工业CAD格式之间的双向转换。这是让AI版型生成研究落地的"最后一公里"基础设施。

## 指向的Gap

- **Gap 4: 跨CAD互操作性** — 不同学术方法和工业工具使用不兼容的版型格式
- **Gap 21: 工业格式输出** — 没有任何学术方法直接输出DXF、ASTM D6673等工业格式

## 推导过程

1. **Textile IR (2026)的启发**：Teikari & Fuenmayor的position paper提出了双向中间表征的概念，但无任何实现——说明学术界已认识到这个问题但尚未解决
2. **工程价值极高**：即使AI版型生成方法完美，如果输出不能导入到Gerber/Lectra/CLO3D中，工厂和设计师就无法使用
3. **开源工具提供基础**：Valentina/Seamly2D等开源CAD的源代码可以作为格式解析的参考

## 详细设计

### 1. 中间表征（Pattern IR）设计

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
      "seam_allowances": {
        "edge_0": {"width": 1.0, "type": "standard"},
        "edge_1": {"width": 0.7, "type": "curved"},
        "edge_2": {"width": 2.0, "type": "hem"}
      },
      "notches": [
        {"edge": 1, "position": 0.35, "type": "V-notch"},
        {"edge": 2, "position": 0.5, "type": "T-notch"}
      ],
      "grain_line": {"start": [5, 5], "end": [5, 18]},
      "internal_marks": [
        {"type": "dart_point", "position": [8, 12]},
        {"type": "button", "position": [10, 5]}
      ]
    }
  ],
  "stitches": [
    {"panel_a": "front_bodice", "edge_a": 1,
     "panel_b": "back_bodice", "edge_b": 0,
     "seam_type": "plain_seam"}
  ],
  "grading": {
    "rules": [...],
    "sizes": [36, 38, 40, 42, 44, 46]
  }
}
```

### 2. 双向转换器

**学术 → IR**：

| 学术格式 | 解析方法 |
|----------|---------|
| GarmentCode程序 | 执行程序 → 提取panel几何和stitch信息 |
| SewingGPT tokens | 解码token序列 → 恢复panel参数 |
| GarmentDiffusion edge tokens | 解码edge矩阵 → 恢复panel轮廓 |
| GarmageNet garmage图像 | 反光栅化 → 提取向量化panel |
| ChatGarment JSON | 直接解析JSON结构 |

**IR → 工业**：

| 工业格式 | 生成方法 |
|----------|---------|
| DXF (AutoCAD) | 生成ENTITIES (LINE, ARC, POLYLINE) + LAYERS (PATTERN, SEAM_ALLOWANCE, NOTCH, GRAIN_LINE) |
| ASTM D6673 | 生成标准化XML/ZIP包含所有版型数据 |
| CLO3D .zprj | 生成CLO3D可导入的项目文件 |
| Optitex .gmt | 生成Optitex格式的版型文件 |
| SVG (通用) | 生成带图层的SVG用于预览和共享 |

### 3. 验证机制

**Round-trip验证**：
```
GarmentCode程序 → IR → DXF → 导入CLO3D → 3D渲染 → 与原始GarmentCode渲染对比
```

**几何精度验证**：
- Panel面积偏差 < 0.1%
- 边长偏差 < 0.5mm
- 曲线拟合误差 (Hausdorff distance) < 1mm

### 4. 开源实现路径

- 使用Python实现核心转换器
- DXF读写：ezdxf库
- SVG读写：svgwrite + svgpathtools
- CLO3D：逆向分析.zprj格式（ZIP包含JSON+binary mesh）
- 提供CLI工具和Python API

## 参考文献分析

1. **Textile IR** (Teikari & Fuenmayor, 2026, arXiv:2601.02792): 提出概念但无实现。本提案是其实现版本
2. **Design2GarmentCode** (CVPR 2025): 展示了Style3D集成但不输出工业格式
3. **DXF格式规范**: 需要ENTITIES、LAYERS、BLOCKS等结构
4. **ASTM D6673**: 定义数字版型数据交换标准格式
5. **Valentina/Seamly2D**: 开源CAD，源代码可参考格式解析

## 五维评分

| 维度 | 分数 | 理由 |
|------|------|------|
| **新颖性** | 6/10 | 格式转换器本身不算学术创新，但设计统一IR并实现全覆盖有工程创新价值 |
| **可行性** | 7/10 | 格式解析和生成都是确定性的工程任务；开源CAD工具的源代码可以参考 |
| **影响力** | 9/10 | 直接打通学术-工业鸿沟，使所有学术方法的输出都可以进入工业流程 |
| **清晰度** | 9/10 | IR设计、转换逻辑、验证方法都可以精确定义 |
| **证据支撑** | 7/10 | Textile IR的position paper验证了需求；DXF等格式规范公开可查 |
| **总分** | **38/50** | |

## 风险与挑战

1. **工业格式的封闭性**：Gerber/Lectra/CLO3D的文件格式部分是私有的，逆向分析可能不完整
2. **曲线精度损失**：不同格式表示曲线的方式不同（Bezier vs arc vs polyline），转换可能有精度损失
3. **语义信息丢失**：学术格式缺少seam allowance/notch/grain line等信息——转换时需要额外推断或用户输入
4. **维护成本**：每当学术界出现新的版型表示或工业软件更新格式，都需要添加新的转换器
