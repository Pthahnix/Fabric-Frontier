# Context Map

## Version Overview

```mermaid
flowchart TB
    V0["version-0/\n(original: 29 gaps, 15 ideas, rebuttals)"]
    V1["version-1/\n(iterated: 35 gaps, 15 ideas)"]
    V0 -->|"rebuttal + research"| V1

    classDef v0 fill:#95a5a6,color:#fff,stroke:#7f8c8d
    classDef v1 fill:#2ecc71,color:#fff,stroke:#27ae60
    class V0 v0
    class V1 v1
```

## Version 0 (Archived)

All v0 files are in `context/version-0/`. Structure:

- `gaps/` — 6 gap direction files (29 gaps total)
- `ideas/` — 15 idea proposals
- `rebuttal/` — 6 gap rebuttals + 15 idea rebuttals
- `2026-03-12-gaps-and-ideas-comprehensive-report.md` — v0 entry point

## Version 1 (Current)

Entry point: `context/version-1/2026-03-13-gaps-and-ideas-comprehensive-report-v1.md`

### Gap-Idea Mapping

```mermaid
flowchart LR

    %% ── v1 Gap files ──
    G1["gaps/01-program-synthesis\n(Gap 1-6, 2 Critical)"]
    G2["gaps/02-multimodal-foundation\n(Gap 5-10, 1 High→升级)"]
    G3["gaps/03-generation-paradigm\n(Gap 9-12, Gap12 replaced)"]
    G4["gaps/04-3d-to-2d-flattening\n(Gap 13-17, +1 new)"]
    G5["gaps/05-manufacturability\n(Gap 17-21, 3 downgraded to Low)"]
    G6["gaps/06-physics-simulation\n(Gap 22-29, 6 downgraded)"]

    %% ── v1 Idea files (sorted by score) ──
    I09["ideas/09-garmentbench\nmodular-eval 【39分】"]
    I10["ideas/10-fabric-aware\npattern-generation 【39分】"]
    I01["ideas/01-garmentcode++\nself-learning-dsl 【38分】"]
    I04["ideas/04-real-world\npattern-dataset 【38分】"]
    I08["ideas/08-zero-waste-pattern\nconstraint-optim 【37分】"]
    I11["ideas/11-ai-neural\npattern-grading 【37分】"]
    I13["ideas/13-cross-cad\npattern-translator 【36分】"]
    I14["ideas/14-stretch-fabric\nnegative-ease 【36分】"]
    I03["ideas/03-generation-nesting\nsimulation-joint 【35分】"]
    I06["ideas/06-dynamic-fit\nevaluation-network 【35分】"]
    I12["ideas/12-pattern-matching\nconstraint-solver 【35分】"]
    I02["ideas/02-sewing-process\nknowledge-graph 【34分】"]
    I15["ideas/15-design-intent\nformal-verification 【34分】"]
    I05["ideas/05-knitting-instruction\nprogram-synthesis 【33分】"]
    I07["ideas/07-multi-layer-garment\njoint-generation 【32分】"]

    %% ── v1 Report ──
    R1["2026-03-13\ncomprehensive-report-v1"]

    %% ── Gap 01 → Ideas ──
    G1 -->|"Gap 1,2,5"| I01
    G1 -->|"Gap 1"| I05
    G1 -->|"Gap 3"| I02
    G1 -->|"Gap 4"| I13

    %% ── Gap 02 → Ideas ──
    G2 -->|"Gap 5"| I04
    G2 -->|"Gap 5"| I09
    G2 -->|"Gap 6"| I07
    G2 -->|"Gap 10"| I10

    %% ── Gap 03 → Ideas ──
    G3 -->|"Gap 9"| I09

    %% ── Gap 04 → Ideas ──
    G4 -->|"Gap 13"| I10
    G4 -->|"Gap 13"| I14
    G4 -->|"Gap 14"| I15
    G4 -->|"Gap 15,16"| I11
    G4 -->|"Gap 17"| I10

    %% ── Gap 05 → Ideas ──
    G5 -->|"Gap 17,18"| I02
    G5 -->|"Gap 19"| I03
    G5 -->|"Gap 19"| I08
    G5 -->|"Gap 19"| I10
    G5 -->|"Gap 19"| I12
    G5 -->|"Gap 19"| I14
    G5 -->|"Gap 20"| I03
    G5 -->|"Gap 20"| I08
    G5 -->|"Gap 20"| I12
    G5 -->|"Gap 21"| I13

    %% ── Gap 06 → Ideas ──
    G6 -->|"Gap 22,23,28"| I06
    G6 -->|"Gap 23"| I10
    G6 -->|"Gap 25"| I07
    G6 -->|"Gap 26"| I03
    G6 -->|"Gap 26"| I15
    G6 -->|"Gap 27"| I04
    G6 -->|"Gap 27"| I09
    G6 -->|"Gap 28"| I15

    %% ── All Ideas → Report ──
    I01 --> R1
    I02 --> R1
    I03 --> R1
    I04 --> R1
    I05 --> R1
    I06 --> R1
    I07 --> R1
    I08 --> R1
    I09 --> R1
    I10 --> R1
    I11 --> R1
    I12 --> R1
    I13 --> R1
    I14 --> R1
    I15 --> R1

    %% ── Uncovered High-severity Gaps ──
    G1 -.-x|"Gap 6: no idea"| UNCOV["⚠ 7 High gaps\nwithout ideas"]
    G2 -.-x|"Gap 7,9: no idea"| UNCOV
    G3 -.-x|"Gap 12: no idea"| UNCOV
    G6 -.-x|"Gap 29: no idea"| UNCOV

    %% ── Styles ──
    classDef gap fill:#e74c3c,color:#fff,stroke:#c0392b
    classDef idea fill:#3498db,color:#fff,stroke:#2980b9
    classDef topidea fill:#2980b9,color:#fff,stroke:#1a5276,stroke-width:3px
    classDef report fill:#2ecc71,color:#fff,stroke:#27ae60
    classDef uncov fill:#f39c12,color:#fff,stroke:#d68910

    class G1,G2,G3,G4,G5,G6 gap
    class I02,I03,I05,I06,I07,I12,I14,I15 idea
    class I01,I04,I08,I09,I10,I11,I13 topidea
    class R1 report
    class UNCOV uncov
```

### v0→v1 Key Changes

| Metric | v0 | v1 |
| ------ | -- | -- |
| Total Gaps | 29 | 35 (+6 new) |
| Critical Gaps | 10 | 2 (-8) |
| Low/Engineering Gaps | 0 | 4 (+4) |
| Idea avg score | 35.3 | 35.7 |
| Idea max score | 42 | 39 (more calibrated) |

**Core shift**: v0 over-classified engineering tasks as Critical research gaps. v1 distinguishes:

- **Research Gaps**: Need new methods/theory (e.g., DSL expressiveness, VLM quantitative reasoning)
- **Cross-domain Integration**: Tech exists elsewhere, needs transfer (e.g., library learning, differentiable nesting)
- **Engineering Tasks**: Deterministic operations with mature tools (e.g., seam allowance, DXF export)
