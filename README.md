# fabric-frontier

AI-driven garment sewing pattern generation research project. Based on a literature survey of 50+ papers (2023-2026), this repository organizes research gaps, ideas, and reference materials for advancing the intersection of artificial intelligence and garment design.

## Research Scope

The project focuses on AI-driven automatic generation of garment sewing patterns, covering six major paradigms:

| Paradigm | Representative Work | Venue |
|----------|-------------------|-------|
| Program Synthesis | Design2GarmentCode, ChatGarment | CVPR 2025 |
| Autoregressive Sequence | DressCode, SewingGPT | SIGGRAPH 2024 |
| Diffusion Models | GarmentDiffusion | IJCAI 2025 |
| Multimodal Foundation Models | AIpparel | CVPR 2025 |
| Image-based Representation | GarmageNet | SIGGRAPH Asia 2025 |
| Differentiable Simulation | DiffAvatar, Inverse Garment | CVPR 2024 |

## Repository Structure

```
garment/
├── README.md
├── CLAUDE.md
├── context/
│   ├── gaps/                  # 6 research gap analyses (29 gaps total)
│   │   ├── 01-program-synthesis-and-parametric-pattern.md
│   │   ├── 02-multimodal-foundation-model.md
│   │   ├── 03-generation-paradigm-comparison.md
│   │   ├── 04-3d-body-to-2d-pattern-flattening.md
│   │   ├── 05-manufacturability-gap.md
│   │   └── 06-physics-simulation-closed-loop-and-systematic-gap.md
│   ├── ideas/                 # 15 research idea proposals
│   │   ├── 01 ~ 15 idea documents
│   │   └── (scored by novelty, feasibility, impact, clarity, evidence)
│   ├── markdown/              # 30+ paper summaries in markdown
│   └── 2026-03-12-gaps-and-ideas-comprehensive-report.md
└── .claude/
```

## Research Gaps (6 Directions, 29 Gaps)

1. **Program Synthesis & Parametric Patterns** - DSL expressiveness, generalization, sewing semantics, cross-CAD interoperability
2. **Multimodal Foundation Models** - Synthetic data bias, multi-layer garments, semantic ambiguity, editing consistency
3. **Generation Paradigm Comparison** - Fair benchmarks, sequence length, topology changes, editing consistency
4. **3D Body to 2D Pattern** - Stretch fabrics, inverse ambiguity, body shape bias, pattern grading
5. **Manufacturability Gap** - Seam allowance, notch/grain lines, fabric constraints, nesting, industrial formats
6. **Physics Simulation & Systematic Issues** - Simulation speed, sim-to-real gap, closed-loop, metric fragmentation

## Top Research Ideas

| Rank | Idea | Score | Key Gaps Addressed |
|------|------|-------|--------------------|
| 1 | GarmentBench Unified Evaluation Benchmark | 42 | 9, 27, 5 |
| 2 | Real-World Pattern Dataset | 41 | 5, 27 |
| 3 | AI Neural Pattern Grading | 38 | 16, 15 |
| 3 | Cross-CAD Pattern Translator | 38 | 4, 21 |
| 5 | Fabric-Aware Pattern Generation | 37 | 19, 13, 23 |

## Key Trends

- VLMs (Vision-Language Models) deeply involved since 2025: ChatGarment, DressWild, NGL-Prompter
- Shift from single-modal to multimodal input: text + image + sketch + 3D scan
- Growing demand for bridging lab demonstrations to industrial deployment
