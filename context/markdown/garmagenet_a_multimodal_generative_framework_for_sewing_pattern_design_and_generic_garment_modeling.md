<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - 2.1. Garment Datasets
    - 2.1.1. Scanning-based datasets
    - 2.1.2. Simulation-based datasets
    - 2.1.3. Sewing pattern datasets
  - 2.2. Learning-based Garment Modeling
    - 2.2.1. Geometric-centric Garment Modeling
    - 2.2.2. Structure-centric garment modeling
      - Vector-quantization based methods
      - Template-based methods
      - Image-based modeling
  - 2.3. Structural Object Modeling
- 3. Overview
- 4. GarmageNet
  - 4.1. The Garmage Representation
  - 4.2. Diffusion-based Garmage Generation
    - 4.2.1. Latent Encoding.
    - 4.2.2. Diffusion Generation
    - 4.2.3. Dealing with Conditions
- 5. Garamge Processing
  - 5.1. Boundary Point Sampling
  - 5.2. Sewing Relation Recovery
  - 5.3. Sewing Pattern Reconstruction
- 6. GarmageSet
  - 6.1. GarmageSet Construction
    - 6.1.1. Data Acquisition
    - 6.1.2. Garment Structure Definition
    - 6.1.3. Data Formation And Multi-modal Augmentation
  - 6.2. Dataset Statistics
- 7. Experiments
  - 7.1. Implementation Details
  - 7.2. Evaluation And Comparison
    - 7.2.1. Cross-Dataset Comparison on Sewing Pattern Quality
    - 7.2.2. Sewing Accuracy Evaluation
    - 7.2.3. Fine‐Grained 3D Initialization Evaluation
    - 7.2.4. 3D Garment Asset Quality
- 8. Applications
  - 8.1. Design Concept to Garment Generation
  - 8.2. Automatic Garment Modeling
  - 8.3. Sewing Pattern Recovery
  - 8.4. Progressive Generation and Editing
- 9. Conclusion
- 10. Limitations and Future Work
  - Lacking body shape and pose variance
  - Handling of interior seams and pocket-like components
  - Multi-layer designs and small-panel handling
  - Future Work
    - Acknowledgements.
- References

## Abstract

Abstract. Realistic digital garment modeling remains a labor-intensive task due to the intricate process of translating 2D sewing patterns into high-fidelity, simulation-ready 3D garments. We introduce GarmageNet , a unified generative framework that automates the creation of 2D sewing patterns, the construction of sewing relationships, and the synthesis of 3D garment initializations compatible with physics-based simulation.
Central to our approach is Garmage , a novel garment representation that encodes each panel as a structured geometry image, effectively bridging the semantic and geometric gap between 2D structural patterns and 3D garment geometries. Followed by GarmageNet , a latent diffusion transformer to synthesize panel-wise geometry images and GarmageJigsaw , a neural module for predicting point-to-point sewing connections along panel contours.
To support training and evaluation, we build GarmageSet , a large-scale dataset comprising 14,801 professionally designed garments with detailed structural and style annotations.
Our method demonstrates versatility and efficacy across multiple application scenarios, including scalable garment generation from multi-modal design concepts (text prompts, sketches, photographs), automatic modeling from raw flat sewing patterns, pattern recovery from unstructured point clouds, and progressive garment editing using conventional instructions, laying the foundation for fully automated, production-ready pipelines in digital fashion. Refer to our project page for open-sourced code and dataset.

## 1. Introduction

Realistic digital clothing plays a pivotal role in entertainment and gaming by enhancing immersion, and in fashion and e-commerce by accelerating product development and reducing costs. Yet, end-to-end 3D garment modeling remains a labor-intensive and technically demanding process, encompassing line-art design, pattern drafting, sewing assignment, 3D initialization (Liu et al., 2024d), and physics-based simulation. With the rapid progress of generative AI in 2D fashion design, a pressing question arises: can these models extend beyond image generation to automate the entire fashion manufacturing pipeline, transforming creative designs into production-ready sewing patterns and simulation-ready 3D garments that faithfully mirror their physical counterparts?

Garments, unlike images with well-established 2D representations, are complex 3D forms constructed from two-dimensional sewing patterns. These patterns are not merely cutting templates but encode the rules by which flat fabrics are assembled and folded into complex 3D forms. Effective garment modeling must therefore balance 3D geometric fidelity, to achieve realistic draping, with 2D structural correctness, to ensure manufacturability. Despite recent advances, learning-based garment modeling remains fundamentally limited by trade-offs between structure and geometry.

*Structure-centric garment modeling methods* operate in the sewing‐pattern domain.
They use auto-regressive or diffusive frameworks to either (i) generate vectorized (He et al., 2024; Nakayama et al., 2024; Liu et al., 2024a; Li et al., 2025b) or rasterized (Tatsukawa et al., 2025) sewing patterns together with edge-wise sewing correspondences and 3D initializations as per-panel rigid-transformation matrices, or (ii) emit high-level parameters (Bian et al., 2024; Guo et al., 2025a) and programs (Zhou et al., 2024) of a parametric pattern-making DSL such as GarmentCode (Korosteleva and Sorkine-Hornung, 2023).
The generated patterns are then draped onto a target avatar using conventional cloth simulation. While this paradigm preserves structural correctness by explicitly producing sewing patterns, it lacks full spatial context and often fails to reproduce fine folds and realistic drape geometry (Figure [2](https://arxiv.org/html/2504.01483v4#S1.F2) (b)).

In contrast, *geometry-centric approaches* (Yu et al., 2025a; Rong et al., 2024; Zhang et al., 2024; Tochilkin et al., 2024; Xiang et al., 2024; Zhao et al., 2025) map multi-modal design inputs directly into a draped 3D garment. They typically employ continuous, optimizable implicit representations (e.g., distance or occupancy fields) to maintain geometric fidelity, while failed to incorporate sewing pattern structure which is inherently discrete and discontinuous (Figure [2](https://arxiv.org/html/2504.01483v4#S1.F2) (c)). Not to mention that garment meshes recovered from those contiguous fields often exhibit suboptimal geometric quality, making it extremely challenging, if not impossible, to recover sewing patterns from those 3D models with UV parameterization (Srinivasan et al., 2025; Yu et al., 2024).

Figure: Figure 2. Trade-offs between structure-centric and geometry-centric approaches. The magic of wearing (a) indicates that the same sewing pattern could lead to various draping statuses, raising the problem of *Structure-centric modeling*, which focuses on sewing pattern structure but fails to ensure draping alignment with the input (b). On the other hand, *Geometry-centric modeling* focuses on draping alignment but fails to preserve structure integrity in UV space (c).
Refer to caption: figs/fwd_vs_bwd.png

These limitations motivate a *foundational garment-modeling framework* that jointly preserves the structural integrity of 2D sewing patterns and the geometric fidelity of 3D drapes.
In response, we present *GarmageNet*, the first unified framework, to our knowledge, that automatically generates 2D sewing patterns, infers sewing relationships, and produces simulation-ready 3D garment initializations from conventional multi-modal inputs.

At its core is *Garmage* (Figure [3](https://arxiv.org/html/2504.01483v4#acmlabel1) (b)), a novel representation that reformulates a garment asset as a structured collection of per-panel geometry images (Gu et al., 2002) in sewing-pattern (UV) space: the alpha channel encodes each panel’s 2D contour, and the RGB channels capture the draped 3D geometry. This design retains the flexibility of standard image formats for seamless integration with image-based algorithms and enables direct reconstruction of simulation-ready 3D garments.

Building on Garmage, *GarmageNet* formulates garment synthesis as a latent diffusion problem. It first learns a compact manifold of admissible panel variations by encoding each cloth piece’s quasi-static 3D geometry and corresponding 2D pattern into a fixed-length latent token. Using this latent space as a strong prior, a diffusion transformer (DiT) learns to assemble tokens into structurally valid garments, yielding a complete Garmage that preserves panel-wise structure while recovering fine-grained cloth-piece geometry.

We then introduce *GarmageJigsaw*, which recovers sewing relations to convert panel-wise Garmages into a simulator-ready seam graph. Unlike edge-pair matching, GarmageJigsaw models sewing relationships as point-to-point correspondences sampled from boundary pixels in the Garmage’s alpha channel, thereby capturing fine-scale folds and pleats. Those correspondences are then consolidated into edge-to-edge constraints via a specially designed heuristic algorithm, ensuring full compatibility with pattern-design and cloth-simulation software.

To further advance GarmageNet toward real-production quality, we present *GarmageSet*, a large-scale dataset comprising 14,801 professionally designed, production-ready(^1^11We define “production-ready” as assets derived from real-world manufacturing data (as opposed to purely synthetic datasets such as GarmentCode), conforming to industry standards and fully compatible with existing garment-production workflows.) garments, each represented as a Garmage with fine-grained structural and style annotations.

In summary, our contributions include:

- •
We propose *GarmageNet*, the first unified system that *simultaneously* produces structurally correct sewing patterns and vertex-level-precise, simulation-ready 3D garments from various input modalities, including textural descriptions, line-art sketches, commercial photos, unstructured point clouds or flat patterns.
- •
We introduce *Garmage*, a novel and compact representation that seamlessly encodes a garment’s discrete sewing pattern structure and continuous draping geometry into fixed-length latent tokens, facilitating efficient integration with diffusion-based generative models and multi-modal cross-attention conditioning.
- •
We develop *GarmageJigsaw* to recover sewing relationships between Garmage panels by predicting point-to-point stitches along panel contours and convert Garmage into production-ready garment assets, facilitating downstream editing of sewing patterns, material properties, and dynamic simulations.
- •
We construct *GarmageSet*, a large-scale dataset of 14,801 professionally-crafted garment assets with industrial-grade sewing patterns originated from real manufacturing workflows, alongside detailed structural and style annotations, advancing learning-based garment modeling toward practical usages.
- •
Cross-dataset evaluations demonstrate that our framework outperforms both structure-centric and geometry-centric baselines and enables practical applications, including scalable multimodal garment generation, sewing-pattern recovery from point clouds, automatic 3D modeling from flat patterns, and interactive editing via intuitive commands.

## 2. Related Work

In traditional physics-based garment modeling, the simulation (Wu et al., 2022; Wang et al., 2018b) accelerated by specialized hardware (He et al., 2025) is only the final step of a complex pipeline. Prior to simulation, practitioners must design 2D patterns, define sewing relationships, select fabric properties (Wang et al., 2023; Feng et al., 2022) and mesh resolutions (Zhang et al., 2025), and carefully initialize the 3D configuration to avoid cloth–body interpenetration and self-collisions (Wang et al., 2015; Tang et al., 2014; Wang et al., 2016, 2018a), and non-physical draping in garments with intricate structures (Guo et al., 2025c, d). Early automation addressed isolated substeps. For example, learning-based sewing identification still required manual initialization and bespoke parsers (Berthouzoz et al., 2013), while recent automatic initialization methods presuppose complete, professionally made patterns with correct seam graphs (Liu et al., 2024d). These constraints have motivated learning-based approaches, which in turn depend on large, high-quality datasets. Below, we review available garment datasets and the two primary modeling paradigms they enable.

### 2.1. Garment Datasets

Learning-based garment modeling draws on three main data sources: scanning-based, simulation-based, and sewing pattern-based.

#### 2.1.1. Scanning-based datasets

They capture the geometry and appearance of clothed humans using multi-view RGB rigs, RGB-D sensors, or structured light (Pons-Moll et al., 2017; Xiang et al., 2020; Zhang et al., 2017; Lin et al., 2023). Pipelines reconstruct per-frame surfaces via multi-view stereo or volumetric fusion (Pons-Moll et al., 2017; Zhang et al., 2017), then non-rigidly register a common topology (e.g., SMPL (Loper et al., 2015) or garment/body templates (Xiu et al., 2023)) to obtain temporally consistent meshes and UV textures (Bhatnagar et al., 2019; Tiwari et al., 2020; Ma et al., 2020; Wang et al., 2024b; Lin et al., 2023). While these datasets excel at capturing realistic drape, wrinkles, and dynamics, isolating semantically meaningful parts from the raw scans remains a labor-intensive process, heavily reliant on manual efforts. As a result, these datasets typically lack sewing patterns that match the garment assets. Additionally, they are mostly derived from existing commercial garment asset libraries, limiting their scale and design diversity.

#### 2.1.2. Simulation-based datasets

Unlike scanning-based datasets that focus on garment drape but typically lack explicit sewing structure, simulation based datasets start from manually authored garment assets and use physics engines to synthesize draping across diverse body shapes and motion sequences  (Bertiche et al., 2020; Black et al., 2023; Gundogdu et al., 2019; Jiang et al., 2020; Patel et al., 2020; Zou et al., 2023; Santesteban et al., 2019; Narain et al., 2012). Because authoring high-quality assets is labor-intensive and these datasets primarily target clothed-human reconstruction, they emphasize automated expansion of pose and appearance variation, while offering limited structural diversity, often only one or two hundreds of distinct garment designs.  (Luo et al., 2024) are among the first to scale structural variety to the thousands (with 2,869 unique designs); however, the assets remain less complex than production garments without considering multilayer constructions, complex seam topologies, and sewing patterns are not provided.

#### 2.1.3. Sewing pattern datasets

Compared with scanning or simulation based dataset, sewing pattern datasets remain relatively nascent. GarmentCodeData (Korosteleva and Lee, 2021; Korosteleva et al., 2024) is the most widely used sewing-pattern dataset in the research community, generated by programmatically sampling parametric sewing-pattern programs (GarmentCode), and draping onto SMPL (Loper et al., 2015) avatars via a fast XPBD simulator (Macklin, 2022). Although its procedural generation enables easy scaling, it substantially simplifies realities of production data. For example, production patterns contain multi-edge-loop panels and intricate contours (e.g., spirals), beyond the single-edge-loop and four edge-type assumptions in GarmentCodeData; In terms of sewing representation, industrial garments require many-to-many, partial-edge mappings to support multi-layering and dense pleats, whereas GarmentCodeData encodes one-to-one full-edge correspondences; and GarmentCodeData ships with per-panel rigid placements that allow immediate draping, while in practice vendors typically provide only flat DXF patterns without seam graphs or rigid placements, making the recovery of simulator-ready initializations a nontrivial task.

### 2.2. Learning-based Garment Modeling

Depending on the garment representations provided by those datasets, learning-based garment modeling methods can be broadly categorized into two families: geometry-centric and structure-centric.

#### 2.2.1. Geometric-centric Garment Modeling

Implicit garment modeling methods often rely on unsigned distance fields (UDF) (Yu et al., 2025a; Chen et al., 2024), manifold distance fields (Liu et al., 2024b), and Gaussian splatting (Liu et al., 2024c; Rong et al., 2024) to handle non-watertight garment geometry, and employ diffusion or GAN-based generative models to generate visually pleasant garment assets, with vivid dynamics (Xie et al., 2024; Rong et al., 2024). However, how to transform those implicit representations into triangular or quadrilateral meshes relies on a solid iso-surface extraction algorithm, which remains quite a challenging problem.

Another option is to explicitly deform a template garment mesh and register it to a posed avatar via end-to-end optimization (Zhu et al., 2022; Qiu et al., 2023) or differentiable simulators (Li et al., 2024a; Sarafianos et al., 2024). While these methods are able to preserve the template’s topological integrity, the geometric expressiveness and complexity of the deformed mesh are also constrained by the template itself.

#### 2.2.2. Structure-centric garment modeling

Contrary to geometry-centric modeling, *structure-centric* approaches first predict sewing patterns and then drape them onto a target avatar with a cloth simulator. Depending on the representation of the sewing pattern, existing methods fall into three families: vector-quantization based, template-based (program or parameter), and image-based encodings.

##### Vector-quantization based methods

The pioneering NeuralTailor (Korosteleva and Lee, 2022) formulates sewing-pattern reconstruction from unstructured point clouds as sequence prediction by vector-quantizing patterns into 1D tokens and decoding them with an LSTM equipped with point-level attention. Subsequent work broadens the input modalities: DressCode (He et al., 2024) generates sewing patterns from text, while SewFormer (Liu et al., 2023) predicts them from images, with (Li et al., 2025c) integrates differentiable simulator to further optimize the predicted patterns so the simulated drape better matches the input image.
Recent diffusion-based (Liu et al., 2023; Li et al., 2025b) and LLM-based (Nakayama et al., 2024) variants further refine the vector-quantized formulation and architectures to scale to larger, more complex GarmentCodeData and to accommodate diverse modalities.

##### Template-based methods

Due to the probabilistic nature of neural-network generators, vector-quantization based methods do not guarantee seam correctness and can produce geometric defects such as self-intersections. To mitigate this, template-based methods (Bian et al., 2024; Zhou et al., 2024; Guo et al., 2025a) exploit the programmatic nature of GarmentCode (Korosteleva and Sorkine-Hornung, 2023) by generating high-level design parameters or DSL programs, which enforces structural validity and improves pattern quality. Nevertheless, both vector-quantization and template-based approaches inherit simplifying assumptions from GarmentCodeData and still require physics simulation to perceive the final on-body drape.

##### Image-based modeling

Our Garmage formulation is informed by image-based sewing-pattern modeling that preserves the 2D nature of patterns using explicit (Chen et al., 2022; Tatsukawa et al., 2025) or implicit (Li et al., 2024d) image-like encodings. Although raster encodings are more redundant than 1D tokenizations, this capacity allows us to embed UV-aligned position maps that depict 3D drape (Li et al., 2024c, b, 2025a) and allowing convenient integration with recent image generative models (Gu et al., 2002; Yan et al., 2024; Yu and Wang, 2024). The added redundancy also necessitates a compact encoding to balance computation and fidelity. We address this with a patch-based geometry-image design that compresses per-panel content while preserving sewing structure, which is a key novelty of Garmage.

Figure: Figure 3. Overview of our *GarmageNet* framework, which seamlessly converts multi-modal design inputs—including text descriptions, sewing patterns, line-art sketches, and point clouds (a)—into simulation-ready garment assets (d). Central to our framework is the novel *Garmage* representation, a 2D–3D unified representation that encodes each garment as a structured set of per-panel geometry images (b). Leveraging *Garmage*, our approach efficiently recovers vertex-level sewing relationships and detailed 3D draping initializations (c), enabling direct and high-quality garment simulation.(^†^†:)
Refer to caption: figs/overview.jpg

### 2.3. Structural Object Modeling

Recent advancements in structural object modeling have enhanced the generation and reconstruction of Boundary Representation (B-rep) models, enabling more complex 3D shape synthesis for CAD applications. BRepGen (Xu et al., 2024) uses a diffusion-based approach to generate B-rep models hierarchically, capturing intricate geometries, while SolidGen (Jayaraman et al., 2022) employs autoregressive neural networks to predict B-rep components with indexed boundary representation, facilitating high-quality CAD model generation. ComplexGen (Guo et al., 2022) detects geometric primitives and their relationships to create structurally faithful CAD models.

StructureNet (Mo et al., 2019), DPA-Net (Yu et al., 2025b), and TreeSBA (Guo et al., 2025b) focus on basic geometric shapes but struggle with non-rigid structures and sewing relationships in garments. StructEdit (Mo et al., 2020) targets local editing of geometric bodies, suitable for regular shapes but limited for flexible, multi-layer garments. 3D Neural Edge Reconstruction (Li et al., 2024e) reconstructs rigid object contours but does not handle flexible garment modeling.

Fracture assembly methods like PuzzleFusion++ (Wang et al., 2024a) and Jigsaw (Lu et al., 2024) infer matching relationships for rigid objects but fail with garments, where misaligned contours and segment-to-segment connections are common. In *Garmage*, we adapt this approach by treating panels as *fractures*, predicting relationships between their contour points to establish sewing connections. Unlike rigid objects, garment contours may not align perfectly, and Garmage addresses this by incorporating garment-specific properties, such as curvature, edge smoothness, and sewing constraints in a learning-based framework.

## 3. Overview

Traditional digital garment modeling demands expert intervention to draft 2D sewing patterns and manually arrange panels around articulated avatars for physics-based simulation. Although learning-based approaches have begun to automate pattern creation, they typically lack explicit 3D geometric guidance, resulting in imprecise outputs and difficulty in handling complex draping behaviors.

We present the first unified learning-based garment creation framework that encodes sewing-pattern structure and its quasi-static drape geometry in a unified image-like representation, *Garmage*.
As shown in Figure [3](https://arxiv.org/html/2504.01483v4#acmlabel1), the system ingests multimodal design inputs, including text descriptions, sketches, scanned point clouds, and raw sewing patterns, and synthesizes Garmages with a diffusion transformer (DiT).

After generation, we extract 2D sewing contours by thresholding the alpha channel of the generated Garmage and reconstruct each cloth panel’s 3D shape by denormalizing the Garmage’s RGB channels with the panel’s bounding box. To recover stitching topology, our GarmageJigsaw module jointly leverages 2D silhouette features and 3D proximity of contour points, producing point correspondences that we fit with Bézier curves to yield manufacturable seam lines.
As a result, our method extracts, from the generated Garmage, the garment’s 2D sewing patterns, sewing relationships, and vertex-wise fine-grained initial 3D geometry positioned around a digital avatar—ready for physics-based simulation.

The effectiveness of our framework depends on a comprehensive, multimodal garment corpus, which we call GarmageSet. To support all input pathways, each entry in GarmageSet pairs a ground-truth Garmage (with per-panel geometry images, dimensions, and bounding boxes) with four aligned modalities: a natural-language description, a line-art sketch, a vectorized sewing pattern, and a uniformly sampled point cloud of the draped garment. GarmageSet contains 14,801 garments across varied styles and categories, and provides the rich supervision necessary to train and evaluate GarmageNet.

## 4. GarmageNet

While intertwining 3D geometry and 2D structure pose challenges for learning-based garment encoding and generation, it confers a unique advantage: garment assets inherently possess well-structured and semantically meaningful UV spaces (i.e., their sewing patterns). Each texel in a sewing-pattern panel simultaneously maps to a point on its corresponding 3D cloth piece, creating a natural bridge between structure and geometry. Garmage exploits this insight by converting each cloth piece into a panel-aligned geometry image, whose color channels encode the piece’s quasi-static 3D geometry and whose alpha channel delineates the panel contour. By harnessing the complementary strengths of 2D and 3D representations, Garmage enables efficient, high-quality 3D garment creation without sacrificing structural fidelity.

### 4.1. The Garmage Representation

Traditionally, a garment asset is represented as a set of 3D cloth pieces $\mathcal{C}=\{C_{i}\}_{i=1}^{N}$ and their corresponding 2D sewing pattern panels $\mathcal{P}=\{P_{i}\}_{i=1}^{N}$, where each panel in the sewing pattern maps directly to a cloth piece in 3D space.

In Garmage, we model a garment as a set of per-panel geometry images (Gu et al., 2002; Sander et al., 2003), each of which simultaneously encodes a panel’s 2D contour and its normalized 3D shape. Formally, we define:

$$ (1) $\mathcal{G}=\bigl\{(P_{i},C_{i})\bigr\}_{i=1}^{N}=\bigl\{(D_{i},B_{i},I_{i})\bigr\}_{i=1}^{N},$ $$

where $D_{i}=(h_{i},w_{i})\in\mathbb{R}^{2}$ gives the $i-$the panel’s physical height and width dimension (in meters) aligned with the fabric’s warp and weft; $B_{i}=(o_{i},s_{i})\in\mathbb{R}^{6}$ specifies the axis-aligned bounding box of cloth piece $C_{i}$, parametrized by its center $o_{i}\in\mathbb{R}^{3}$ and half-extents $s_{i}\in\mathbb{R}^{3}$;
and $I_{i}\in\mathbb{R}^{H\times W\times 4}$ is a 4-channel image patch whose first three channels encode $C_{i}$’s geometry normalized by $B_{i}$ and whose alpha channel delineates the panel contour as an occupancy map.

To construct each image patch $I_{i}$, we rasterize the cloth piece $C_{i}$ under its panel $P_{i}$’s UV parameterization at a uniform resolution $(H=W=256)$. Before rasterization, we rotate the 2D panel $P_{i}$ so its warp direction aligns with the $v^{+}$ axis, then normalize its coordinates to $[-1,1]$ by its physical dimension $D_{i}=(h_{i},w_{i})$. Simultaneously, we map every 3D vertex $v_{j}\in C_{i}$ into normalized space via $(v_{j}-o_{i})/s_{i}\in[-1,1]^{3}$ using its bounding box $B_{i}=(o_{i},s_{i})$. At each pixel center $u_{p}\in[-1,1]^{2}$ (corresponding to a 3D point $p$), we test whether $p$ falls inside any triangle of the cloth piece; if so, we find the containing triangle’s vertices $j$ and their barycentric weights $\beta_{j}(p)\in\mathbb{R}^{3}$ to $p$, then set the pixel value as the sum of the normalized position of those vertices weighted by the barycentric weights:

$$ (2) $I_{i}(u_{p})=\begin{cases}\Bigl(\sum_{j}\beta_{j}(u_{p})\frac{v_{j}-o_{i}}{s_{i}},\,&1\Bigr),\quad p\in C_{i},\\ \Bigl((0,0,0),&0\Bigr),\quad\text{otherwise.}\\ \end{cases}$ $$

Rasterizing panels with sharp features (e.g., dart tips) requires careful handling of boundary aliasing. Following (Yan et al., 2024), we run the rasterization at an initial high $1024\times 1024$ resolution and subsequently downsample to the target resolution $256\times 256$ via sparse pooling.

Figure: Figure 4. Overview of our *GarmageNet* architecture. During the *geometry encoding* stage (top), each garment is encoded into a set of fixed-size (72-dimensional) latent vectors using a Variational Autoencoder (VAE). These compact latent representations serve as training targets for the subsequent *diffusion generation* stage (bottom). In the diffusion stage, we employ a diffusion transformer (DiT) denoiser, integrating multi-modal conditions, including line-art sketches, raw sewing patterns, and point clouds via cross-attention mechanisms to effectively guide and control the garment generation process.(^†^†:)
Refer to caption: figs/garmagenet.png

### 4.2. Diffusion-based Garmage Generation

Garment panels serving similar functions often exhibit similar shape features and consistent spatial relationships relative to the human body. For instance, bodice panels typically feature characteristic structural elements such as necklines and armholes and are generally positioned over the chest region in 3D space. This regularity implies a strong correlation between a panel’s silhouette (encoded in the alpha channel of $I_{i}$) and the geometry status of its corresponding cloth piece (the remaining channels in $I_{i}$), motivating the compression of Garmages into a unified latent space that simultaneously captures both silhouette and geometry.

#### 4.2.1. Latent Encoding.

Consider the geometry-image component $I_{i}$ of the $i-$th panel in a Garamge $\mathcal{G}$, we leverage UNet-based variational autoencoder $\bigl(\Phi_{\mathcal{E}}(\cdot),\Phi_{\mathcal{D}}(\cdot)\bigr)$, to compress $I_{i}$ into a 64-dimensional latent vector:

$$ (3) $\begin{split}\Phi_{\mathcal{E}}:&\,\mathbb{R}^{256\times 256\times 4}\to\mathbb{R}^{64},\quad\Phi_{\mathcal{E}}(I_{i})=Z_{i},\\ \Phi_{\mathcal{D}}:&\,\mathbb{R}^{64}\to\mathbb{R}^{256\times 256\times 4},\quad\Phi_{\mathcal{D}}(Z_{i})\approx I_{i}.\end{split}$ $$

To further reinforce 2D–3D correlation in the latent space, during training, we randomly mask out the geometry channels of $I_{i}$ with probability $0.25$ before passing it through the encoder $\Phi_{\mathcal{E}}(\cdot)$, and forcing the decoder $\Phi_{\mathcal{D}}(\cdot)$ to reconstruct the geometry part solely from the panel’s silhouette during inference. This masking scheme also enables flexible Garmage generation from raw sewing patterns.

After latent compression, any garment can be represented as a set of fixed-length latent tokens:

$$ (4) $\mathcal{G}=\mathcal{T}=\bigl\{T_{i}\bigr\}_{i=1}^{N}=\bigl\{(D_{i}\oplus B_{i}\oplus Z_{i})\bigr\}_{i=1}^{N}\in\mathbb{R}^{N\times 72},$ $$

where $N$ represents the number of panels in each garment, $D_{i}\in\mathbb{R}^{2}$ represents the 2D physical dimension of the panel, $B_{i}\in\mathbb{R}^{6}$ is the axis-aligned bounding box of its corresponding cloth piece and $Z_{i}\in\mathbb{R}^{64}$ is the geometry latent.

We train the autoencoder with MSE loss to minimize the reconstruction error, along with a low-weighted ($\lambda_{reg}=1e-6$) KL divergence term between the encoder’s approximate posterior $q_{\Phi_{E}}(z|I)$ and a standard normal distribution $p(z)=\mathcal{N}(0,1)$:

$$ (5) $\mathcal{L}_{enc}=\frac{1}{N}\sum_{i=1}^{N}\|I_{i}-\Phi_{\mathcal{D}}(z_{i})\|_{2}^{2}\,+\,\lambda_{reg}D_{KL}\bigl[q_{\Phi_{\mathcal{E}}}(z|I_{i})\,\|\,p(z)\bigr].$ $$

#### 4.2.2. Diffusion Generation

Based on the learned Garmage latent space, we train a diffusion transformer (DiT) to map random samples from the standard normal distribution $\epsilon\backsim\mathcal{N}(0,1)$ to valid Garmages based on various user input conditions $c$. Specifically, in the forward process, we gradually interpolate the input token $\mathcal{T}$ with random noise through $0\leq t\leq 1000$ timesteps turning it into noisy states $\mathcal{T}_{t}=\sqrt{\alpha_{t}}\mathcal{T}_{0}+\sqrt{1-\alpha_{t}}\epsilon_{t}$ at each timestep. In the backward process, we linearly embeds the noised latent $\{Z_{i,t}\}_{i=1}^{N}$ into patch tokens, embeds the 2D dimension and 3D bounding box $\{D_{i,t}\oplus B_{i,t}\}_{i=1}^{N}$ into position tokens, and add them together with the embedded timesteps to construct the noisy state, and train the diffusion transformer $\Psi_{\mathcal{G}}(\cdot)$ to recover the added noise from the previous timestep $t-1$ to $t$, conditioned on the input condition $c$:

$$ (6) $\begin{split}&\Psi_{\mathcal{G}}:\mathbb{R}^{N\times 72}\to\mathbb{R}^{N\times 72},\quad\Psi_{\mathcal{G}}(\mathcal{T}_{t},t,c)\approx\epsilon_{t},\\ &\Psi_{\mathcal{G}}(\mathcal{T}_{t},t,c)=\textbf{DiT}\Bigl(\textbf{PosEmb}(\mathbf{D}_{t}\oplus\mathbf{B}_{t})+\textbf{MLP}(\mathbf{Z}_{t}),\,\,t,c\Bigr),\\ &\mathbf{D}_{t}\oplus\mathbf{B}_{t}=\{D_{i,t}\oplus B_{i,t}\}_{i=1}^{N}\in\mathbb{R}^{N\times 8},\;\mathbf{Z}_{t}=\{Z_{i,t}\}_{i=1}^{N}\in\mathbb{R}^{N\times 64}.\end{split}$ $$

We train $\Psi_{\mathcal{G}}(\cdot)$ using mean-squared error between the predicted noise $\Psi_{\mathcal{G}}(\mathcal{T}_{0},c,t)$ and added noise $\epsilon_{t}$:

$$ (7) $\mathcal{L}_{\Psi}=\mathbb{E}_{t,\mathcal{T}_{0},\epsilon_{t}}\Bigl[\|\epsilon-\Psi_{\mathcal{G}}(\mathcal{T}_{0},c,t)\|_{2}^{2}\Bigr],$ $$

where $\epsilon_{t}$ denotes the Gaussian noise added at timestep $t$.

All panels in a Garmage are denoised in parallel while the self-attention mechanism of the Transformer backbone implicitly models the connections between panels, ensuring structural validity of the generated garment. For convenience, we zero‑pad each garment to have a fixed number of panels $N=32$ during training and discarding any panels whose bounding box volume $|B_{i}|<0.075$ or 2D dimension $\|D_{i}\|^{2}<1e^{-4}$ at inference time to accommodate panel number variance.

#### 4.2.3. Dealing with Conditions

During diffusion training, each design modality is first encoded into its own latent space by a pretrained encoder and then injected into the Garmage denoiser via cross-attention.
*Text prompts* are mapped to a 1024-dimensional text latent using the CLIP text encoder;
*Line-art sketches* are passed through a pretrained DINOv2 vision transformer, also yielding a 1024-dimensional image latent;
and *unstructured point clouds* are processed by a PointTransformer v3 (PTv3) fine-tuned on GarmageSet for panel segmentation tasks, producing a 1024-dimensional point latent (Wu et al., 2024).

Unlike other modalities, raw sewing pattern conditioned generation is natively supported by GarmageNet via our VAE’s masking scheme (Section [4.2.1](https://arxiv.org/html/2504.01483v4#S4.SS2.SSS1)), which allows for inferring the full 4-channel geometry images from the silhouettes alone.
Consequently, when a *sewing pattern* is provided, its 2D dimensions $\mathbf{D}_{0}$ and geometry latents $\mathbf{Z}_{0}$ are known a prior, leaving only the 3D bounding‐boxes $\mathbf{B}_{0}$ to be recovered via diffusion. In practice, at each diffusion step $t$, we corrupt $\mathbf{D}_{0}$ and $\mathbf{Z}_{0}$ to their noised version $\mathbf{D}_{t}$, $\mathbf{Z}_{t}$ at timestep $t$, concatenate them with $\mathbf{B}_{t}$, and allow the network to iteratively denoise $\mathbf{B}_{t}$ toward the desired 3D bounding-boxes $\mathbf{B}_{0}$.

It is worth noting that although Garmage builds on geometry images, our panel-wise geometry-image design enables more efficient latent compression. Combined with the GarmageNet architecture, which explicitly models inter-panel spatial and connectivity relations, the framework achieves substantial gains in generation quality and efficiency compared with the multi-chart geometry-image baseline Omage (Yan et al., 2024) (Table [4](https://arxiv.org/html/2504.01483v4#S7.T4)).

Figure: Figure 5. Illustration of stitch representation ambiguity and our point‐wise solution. Existing edge‐based methods suffer from inconsistencies due to arbitrary edge splits: in (a) and (b), the red lines depict the same physical stitch, yet their extracted edge features (shown below) differ in both length and parameter encoding. In contrast, our point‐wise stitching (c) directly anchors stitch correspondences to mesh vertices in physical space, producing consistent, robust sewing relationships independent of panel tessellation.
Refer to caption: x2.png

## 5. Garamge Processing

With GarmageNet, we can synthesize complete Garmages from conventional design input, the next challenge is to integrate these rasterized panel images into traditional garment‐modeling pipelines, which requires vectorizing panel contours and reestablishing sewing relationships.
In the following section, we introduce GarmageJigsaw, a dedicated module that leverages Garmage’s embedded 2D silhouettes and 3D spatial cues to robustly infer vertex-wise sewing correspondences, followed by post‐processing routines that yield production‐ready, vector‐format sewing patterns.

### 5.1. Boundary Point Sampling

Conventional vector-quantization based garment modeling systems (Sec.  [2.2.2](https://arxiv.org/html/2504.01483v4#S2.SS2.SSS2)) define sewing relationships as continuous curve-to-curve correspondences. While straightforward, this edge-based scheme often introduces ambiguity due to ill-defined edge separation and complex many-to-many mappings (Figure [5](https://arxiv.org/html/2504.01483v4#S4.F5)), which can cause vector-quantization based methods to fail to converge (see Exp.2 in Table [2](https://arxiv.org/html/2504.01483v4#S7.T2)).
Therefore, we instead represent sewing as connectivity between boundary vertices of cloth pieces, an atomic stitch representation that is not subject to further subdivision.

As discussed before, in Garmage, each panel is represented by a four-channel image patch $I_{i}\in\mathbb{R}^{256\times 256\times 4}$, whose alpha channel $[I_{i}]_{4}$ delineates the panel contour. We extract the set of 2D contour points as:

$$ (8) $\begin{split}&\partial I_{i}\in\mathbb{R}^{k_{i}\times 2},\text{ and }\\ &\partial I_{i}=\bigl\{\,u_{p}:[I_{i}]_{4}(u_{p})>0\bigr\}\;\setminus\;\bigl\{\,u:\bigl([I_{i}]_{4}\ominus\Lambda\bigr)(u_{p})>0\bigr\},\end{split}$ $$

where $\ominus\Lambda$ denotes binary erosion with elliptical structuring element $\Lambda$ of size $(3,3)$, and $k_{i}$ refers to the number of contour points from image patch $I_{i}$.
Denoting $[I_{i}]_{0:3}$ as the remaining geometric channels from $I_{i}$, we retrieve the corresponding 3D points for $\partial I_{i}$ from $[I_{i}]_{0:3}$ and denormalize them into world coordinate with the panel’s corresponding bounding box $B_{i}$:

$$ (9) $\rho I_{i}\in\mathbb{R}^{k_{i}\times 3},\,\,\rho I_{i}=\textbf{Denorm}\bigl(\,[I_{i}]_{0:3}(\partial I_{i}),\,\,B_{i}\,\bigr).$ $$

Note that panels may yield nonuniform point densities, we apply resampling under predefined particle distance to $\rho I_{i}$, ensuring that adjacent contour samples across all panels exhibit consistent 3D distances, producing normalized inputs for our *GarmageJigsaw* correspondence module.

Figure: Figure 6. Overview of sewing relationship recovery and simulation-ready sewing pattern reconstruction from the generated *Garmage* (a). Unlike previous edge-based methods, we predict vertex-level sewing relationships. Specifically, we first sample boundary points (c) from the generated Garmage representation. Our *GarmageJigsaw* takes the boundary points as input, and leverages a point classifier to identify sewing versus non-sewing points (d), followed by a stitch predictor that recovers point-to-point stitches (e), represented as an adjacency matrix. Concurrently, we extract vectorized sewing patterns (b) from the Garmage and transfer the predicted point stitches onto these vectorized patterns (f). We then reconstruct triangle meshes from the vectorized sewing pattern with a Delaunay triangulation constraint by the predicted stitches. Finally, we retrieve vertex-wise draping status from the generated Garmage, leading to a simulation-ready triangle mesh that can be directly integrated into any conventional cloth simulation engine to produce the physically plausible garment (g).(^†^†:)
Refer to caption: figs/garmagejigsaw.png

### 5.2. Sewing Relation Recovery

With the resampled contour points, *GarmageJigsaw* recovers point‐to‐point sewing by jointly leveraging 2D silhouette and 3D geometric features. As shown in Figure  [6](https://arxiv.org/html/2504.01483v4#acmlabel3), we first extract per‐point features using two PointNet++ encoders, $\Phi_{\rho}(\cdot)$ on the 3D contour points $\rho\mathbf{I}\in\mathbb{R}^{K\times 3}$ and $\Phi_{\partial}(\cdot)$ on the 2D pixels $\partial\mathbf{I}\in\mathbb{R}^{K\times 2}$. These features are concatenated and fused through a series of point‐transformer blocks $\Psi_{p}(\cdot)$ to yield a $128$-dimensional per-point feature matrix:

$$ (10) $\begin{split}&f\in\mathbb{R}^{K\times 128},\quad f=\Psi_{p}\Bigl(\,\Phi_{\rho}(\rho\mathbf{I})\,\oplus\,\Phi_{\partial}(\partial\mathbf{I})\,\Bigr),\\ &\rho\mathbf{I}=\{\rho I_{i}\}_{i=1}^{N}\in\mathbb{R}^{K\times 3}\;\text{and}\;\partial\mathbf{I}=\{\partial I_{i}\}_{i=1}^{N}\in\mathbb{R}^{K\times 2}.\end{split}$ $$

Here, $K=\sum_{i}k_{i}$ is the total number of contour points across all panels. A point classifier head $\Phi_{\mathrm{cls}}(\cdot)$ then selects the subset $f^{+}\in\mathbb{R}^{K^{+}\times 128}$ of candidate sewing points by predicting sewing probability based on the point features $f$.

Seam correspondence between two panels is governed by both 3D positional concordance and 2D geometric complementarity (Figure [7](https://arxiv.org/html/2504.01483v4#S5.F7)). To capture both aspects, we follow the primal/dual disentanglement in Jigsaw (Lu et al., 2024), and attach two lightweight MLP heads, $\Phi_{\mathrm{prime}}(\cdot)$ and $\Phi_{\mathrm{dual}}(\cdot)$, to produce for each boundary point a pair of embeddings that disentangle the from-panel view from its complementary counterpart,thereby suppressing self-matches and favoring complementary correspondences:

$$ (11) $\begin{split}&f^{+}_{\mathrm{prime}}\in\mathbb{R}^{K^{+}\times 128},\,\,f^{+}_{\mathrm{prime}}=\Phi_{\mathrm{prime}}(f^{+}),\\ &f^{+}_{\mathrm{dual}}\in\mathbb{R}^{K^{+}\times 128},\,\,f^{+}_{\mathrm{dual}}=\Phi_{\mathrm{dual}}(f^{+}),\end{split}$ $$

We then combine the primal/dual feature with a learnable symmetric weight matrix $\Lambda_{\mathbf{A}}\in\mathbb{R}^{128\times 128}$, followed by a Sinkhorn normalization (Cuturi, 2013) to produce the adjacency probability matrix:

$$ (12) $\mathbf{A}\;=\;\textbf{Sinkhorn}\biggl(\exp\Bigl(\frac{(f^{+}_{\mathrm{prime}})^{\top}\;\Lambda_{\mathbf{A}}\;f^{+}_{\mathrm{dual}}}{\tau}\Bigr)\biggr)\;\in\;[0,1]^{K^{+}\times K^{+}}.$ $$

where $A_{ij}\approx A_{j,i}$ denotes the probability of a sewing exists between the $i$-th and $j$-th contour points, and $\tau$ is a temperature parameter according to  (Lu et al., 2024). The probability matrix $\mathbf{A}$ is processed with the Hungarian algorithm (Fischler and Bolles, 1981), yielding the final point‐to‐point correspondences for seam reconstruction.

The entire GarmageJigsaw model is trained end-to-end with two complementary loss terms: a binary cross-entropy loss $\mathcal{L}_{\mathrm{cls}}$ that supervises the predicted sewing-point probabilities against ground-truth labels $y_{i}\in\{0,1\}$, and a matching loss $\mathcal{L}_{\mathrm{match}}$ that aligns the predicted adjacency matrix $\mathbf{A}$ with the ground-truth matrix $\mathbf{A}_{gt}$. Notably, to prevent the network from trivially minimizing $\mathcal{L}_{\mathrm{match}}$ by omitting sewing pairs, we pad the ground-truth adjacency matrix $\mathbf{A}_{gt}$ into a $K\times K$ with zero columns and rows corresponding to non-sewing points, and compute the matching loss on the whole contour points set $\bigl(\rho\mathbf{I},\partial\mathbf{I}\bigr)$.

Figure: Figure 7. Positional concordance and geometric complementarity of stitched edge pairs. At the armhole, the bodice armscye curve ($\mathbf{AS}$) and the sleeve cap curve ($\mathbf{SC}$) lie in nearly the same 3D location on the draped garment (a), but in 2D sewing pattern space they are complementary in shape: The armscye curve $\hat{\mathbf{AS}}$ is predominantly concave while the sleeve cap curve ($\hat{\mathbf{SC}}$) is convex. Moreover, their stitch orientation opposes each panel’s edge-winding direction: $\hat{\mathbf{SC}}$ is traced with the sleeve panel edge direction, whereas $\hat{\mathbf{AS}}$ is traced against the bodice panel edge direction.
Refer to caption: figs/primal_dual_1.png

We train GarmageJigsaw with vertex-wise seam supervision, where each stitch is a tuple of vertex indices $(v_{a},v_{b})$ (Sec.[6.1.3](https://arxiv.org/html/2504.01483v4#S6.SS1.SSS3)). In the ground-truth assets, stitched vertex pairs are exactly coincident, yielding zero 3D Euclidean distance. By contrast, GarmageNet generated Garmages can exhibit small seam gaps (Figure[3](https://arxiv.org/html/2504.01483v4#acmlabel1) (c)) due to residual noise in panel position and scale left by diffusion denoising, as well as the erosion used during boundary extraction (Eq. [8](https://arxiv.org/html/2504.01483v4#S5.E8)). To improve robustness to such gaps, we apply the following augmentations to the ground-truth sewn vertex pairs during training: Firstly, we apply slight translations and scalings to each panel in both 3D space and 2D sewing-pattern space; then inwardly offset boundary facets by a random amount between 2 and 8 mm; and finally add anisotropic noise aligned with seam directions at stitched boundary vertices and isotropic noise at non-stitched boundary vertices.

Figure: Figure 8. Design space for GarmentSet. Each garment in GarmageSet is annotated along nine professionally defined design dimensions, including *silhouette* (8 options), *darts/pleats* (7), *waistline* (7), *hemline* (7), *neckline* (28), *collar* (17), *opening* (5), *shoulder* (6), and *sleeve* (23). Except for silhouettes, most of those design dimensions have a “/” option indicating that a particular dimension does not apply to the given garment (e.g., sleeve types are irrelevant for skirts or pants).
Refer to caption: figs/garmageset_design_space.png

Figure: Figure 9. Overview of our *GarmageSet* construction process. We first build a component library (b) by structuring sewing patterns collected in the wild (a). Professional modelers then randomly select several components from this library, apply design modifications such as adjusting width, length, or adding decorative details, and assemble them to create diverse, composed 3D garments (c). This approach enables efficient construction of a high-quality dataset capturing extensive design variability and structural complexity.
Refer to caption: figs/data_construction.jpg

### 5.3. Sewing Pattern Reconstruction

In conventional garment-modeling workflows, sewing patterns are represented as vectorized curves, with sewing relationships explicitly defined as pairs between these curve segments. To integrate Garmage-generated results seamlessly into existing pipelines, we must vectorize the panel contours and convert the predicted point-to-point sewings into curve-to-curve correspondences.

To vectorize the Garmage panel contours, we first detect corner points exhibiting sharp turning angles along the contour point set $\partial\mathbf{I}$ by employing a specially designed 1D convolutional filter. We then fit piecewise B-spline curves to contour points between adjacent corners, resulting in smooth, compact vector representations for each panel. This vectorization process effectively smooths slanted boundaries (e.g., the last panel of the $4$-th garment in Figure [1](https://arxiv.org/html/2504.01483v4#S0.F1)) and fills small noisy holes (e.g., the $2$-nd panel of the $5$-th garment in Figure [1](https://arxiv.org/html/2504.01483v4#S0.F1)). Subsequently, we employ a heuristic algorithm to cluster point-to-point stitches predicted by GarmageJigsaw into curve-level sewing correspondences directly on these vectorized B-spline segments.

Finally, we triangulate each sewing-pattern panel into a mesh using constrained Delaunay triangulation (Rognant et al., 1999), guided by the vectorized panel contours and inferred sewing relationships. Specifically, the boundary facets of each cloth piece mesh consist of contour points uniformly resampled according to the sewing correspondences, ensuring smooth and well-aligned seams between adjacent panels.

Vertex positions for these triangulated meshes are determined by sampling the corresponding 3D coordinates from their associated Garmage geometry images using bilinear interpolation, resulting in a fine-grained initial draping state.
In contrast to existing garment modeling frameworks that typically rely on coarse rigid transformations to position each panel, our Garmage-based approach provides vertex-level precision in the initial 3D placement. This capability allows us to accurately capture intricate folding behaviors and nuanced garment structures.

## 6. GarmageSet

As noted, Garmage’s vertex-level sewing and precise 3D initialization excel at modeling intricate drapes and folds, whereas existing datasets (Korosteleva et al., 2024; Luo et al., 2024; Zhu et al., 2020) are restricted to simple, flat garments and cannot fully evaluate our framework. To address this gap, we assembled a professionally curated, industrial-grade dataset *GarmageSet* showcasing complex folding behaviors and multi-layer structures, complete with manually validated structural and style annotations, as well as multimodal augmentations including line-art sketches and sampled point clouds.

### 6.1. GarmageSet Construction

GarmageSet comprises $N=14{,}801$ unique garments spanning five major clothing categories: tops, pants, skirts, dresses, and outerwear. All garments are draped onto an A-posed standard avatar(^2^22Size S mannequin with Asian size 84.) to diminish the geometric variance brought by body sizes and poses.

#### 6.1.1. Data Acquisition

Building *GarmageSet* entirely by hand would be prohibitively time-consuming. To scale the dataset construction efficiently, we adopt a component-centric strategy inspired by GarmentCode (Korosteleva and Sorkine-Hornung, 2023). As illustrated in Figure [9](https://arxiv.org/html/2504.01483v4#S5.F9), we first construct a structured component library from in-the-wild sewing patterns and then task professional modelers with assembling garments by randomly selecting components, applying design modifications (e.g., adjusting width or length, or adding decorative features), and combining them into complete garments.

To build the component library, we collect a diverse set of raw sewing patterns and engage professional pattern makers to annotate them following the hierarchical garment structure definitions detailed in Section [6.1.2](https://arxiv.org/html/2504.01483v4#S6.SS1.SSS2). This process yields a well-organized collection of reusable garment parts, categorized by role (e.g., bodice, sleeve, collar) and tagged with stylistic attributes curated by experienced fashion designers and pattern makers(^3^33Some style tags and illustrations are adapted from Fashionpedia (Fashionary, 2016).), as shown in Figure [8](https://arxiv.org/html/2504.01483v4#S5.F8).

We then randomly sample valid combinations of components from the library and use QWen3 to propose $1$–$3$ modification instructions for each combination, such as altering silhouette proportions or adding style-specific elements. These modified configurations are assigned to professional garment modelers, who manually implement the design changes and assemble the components into finalized 3D garments.

#### 6.1.2. Garment Structure Definition

As illustrated in Figure [10](https://arxiv.org/html/2504.01483v4#S6.F10), garments exhibit a hierarchical structure comprising panels, edges, and landmarks, each capturing distinct semantic and geometric characteristics essential for garment design and construction. To accurately represent and leverage these hierarchical details, we introduce a structured annotation scheme that clearly defines panel-level semantics, structural lines, and fashion landmarks, as described below.

Figure: Figure 10. Garment structure definition and corresponding visualization on both sewing pattern space and 3D garments. (a) Color-coded definitions of eight structural (e.g., body front, sleeve) and seven decorative (e.g., pocket, ruffle) panel classes. (b) A sewing-pattern layout annotated by these semantic labels. (c) The corresponding 3D draped garment on the standard avatar, with each panel rendered according to its semantic class.
Refer to caption: figs/garment_structure.jpg

**Table 1. Panel counts ($\#$Panels) and mean average precision (AP) for semantic segmentation by our fine‐tuned PointTransformer v3, used to derive point‐cloud embeddings for conditional Garmage synthesis. The uniformly high AP values across all categories confirm the model’s robustness in extracting panel‐level semantics from unstructured point clouds, thereby providing a reliable conditioning signal.**
| Category | collar | sleeve | body front | body back | body side |
| --- | --- | --- | --- | --- | --- |
| #Panels | 9807 | 22576 | 34138 | 22608 | 2857 |
| AP | 0.95 | 0.97 | 0.94 | 0.94 | 0.40 |
| Category | skirt/pant front | skirt/pant back | skirt/pant side | hat | stripe |
| #Panels | 28782 | 25242 | 3491 | 2965 | 1142 |
| AP | 0.93 | 0.95 | 0.40 | 0.98 | 0.69 |
| Category | cuff | waist | hem | pocket | ruffles |
| #Panels | 9509 | 16880 | 3088 | 15571 | 2498 |
| AP | *0.98* | 0.96 | 0.79 | 0.93 | 0.42 |

*Panel-level semantics* are established by professional pattern makers based on panel shape, functional role, and placement relative to the human body. As shown in Figure [10](https://arxiv.org/html/2504.01483v4#S6.F10), we identify eight structural classes—collar, sleeve, body front, body back, body side, skirt/pant front, skirt/pant back, and skirt/pant side—as well as seven decorative classes—hat, stripe, cuff, waist, hem, pocket, and ruffles. Annotators assign these semantic labels directly on the 2D sewing patterns using a customized LabelStudio (Tkachenko et al., 2025) annotation tool. Table [1](https://arxiv.org/html/2504.01483v4#S6.T1) reports per-category panel counts in GarmageSet and the average precision (AP) of a panel classifier trained on it. Meanwhile, we use these panel-level semantics to fine-tune a Point Transformer v3 as the point-cloud feature encoder, providing semantic priors for point-cloud–conditioned Garmage generation.

Utilizing per-panel semantic annotations, we extract five types of *structural lines* that define interfaces between semantic panel groups. The neckline delineates boundaries between front/back bodice and collar panels; armholes separate bodice panels from sleeve panels; waistline defines the interface between waist panels and adjacent bodice or skirt/pant panels (or directly between bodice and skirt/pant panels if waist panels are absent); wristline marks the junction between sleeves and cuffs or the lower edge of sleeves if cuffs are absent; and hemline represents the boundary between bodice/skirt/pant panels and hem panels, or the lower edge of these panels when hem panels do not exist. During training, we augment our dataset by perturbing these structural lines, simulating realistic variations in sleeve length, garment length, waist height, and other key design parameters.

Figure: Figure 11. Dataset statistics comparing *GarmageSet* and *GarmentCodeData* (Korosteleva et al., 2024). Histograms illustrate (a) panels per garment, (b) edges per panel, (c) per garment, (d) mesh vertices per garment, and (e) mesh faces per garment distribution among the $10,000$ sampled garments (or $10,000$ panels) from both datasets. Dashed lines indicate the mean and standard deviation for each distribution. GarmageSet exhibits higher average values and broader variance across all metrics, indicating enhanced structural complexity and superior drape fidelity.
Refer to caption: x3.png

*Fashion landmarks* serve as critical reference points for precise garment construction and fitting. Examples include the shoulder tip (SH), bust point (BP), center front neck (CFN), and center front waist (CFW). These landmarks are annotated on both the 2D sewing patterns and their corresponding 3D models using consistent vertex IDs on the mesh. Such dual annotations help align sewing patterns from different garments into a standardized 2D space, eliminating positional ambiguity and facilitating more effective learning. Additionally, these landmarks are consistently projected onto multi-view 2D images, significantly enriching existing fashion landmark datasets and improving the accuracy of fashion landmark estimation and retrieval models, ultimately offering comprehensive support for diverse fashion AI applications.

#### 6.1.3. Data Formation And Multi-modal Augmentation

For each garment, we partition its raw 2D patterns into individual panels $P_{i}$ and compute their physical dimensions $\mathbf{d}_{i}$ and axis-aligned bounding boxes $\mathbf{B}_{i}$. We then rasterize each cloth piece’s normalized 3D mesh $C_{i}$ into a $256\times 256\times 4$ geometry image $I_{i}$, and construct the Garmage representation for the garment (Section [4.1](https://arxiv.org/html/2504.01483v4#S4.SS1)).

The ground-truth *sewing information* is stored as vertex–vertex pairs $(v_{a}^{i},v_{b}^{j})$ from cloth pieces $C_{i}$ and $C_{j}$. During Garmage rasterization, each vertex $v_{a}^{i}$ is projected to a 2D pixel coordinate $u_{a}^{i}\in[0,1]^{2}$. We encode panel identity by shifting the integer part of the pixel coordinate with the panel index, and thus reformulate seams in a panel–pixel domain of shape $M\times 2\times 2$:

$$ (13) $\begin{split}&\mathcal{S}=\{s_{k}\}_{k=1}^{M}\in\mathbb{R}^{M\times 2\times 2},\;\;\text{ where}\\ &s_{k}=(i+u_{a}^{i},\,j+u_{b}^{j})=\bigl(\emph{Rast}(v_{a}^{i},C_{i}),\,\emph{Rast}(v_{b}^{j},C_{j})\bigr).\end{split}$ $$

To train GarmageNet under *multi-modal conditions*, we align each Garmage $\mathcal{G}$ with four modalities:

- •
A manually annotated *short sentence* captures each garment’s category, silhouette, and design details according to a set of professionally defined dimensions (Figure  [8](https://arxiv.org/html/2504.01483v4#S5.F8)). During modeling, we ask the designers to label all applicable dimensions for a given garment asset and leverage Qwen3 (Yang et al., 2025) to reformat the annotation as a CLIP-compatible, comma-separated string, with the first segment always denoting the garment category. During training, we randomly delete at most $4$ design detail descriptions.
- •
A set of *line-art sketches and clay renderings* to capture each garment’s visual characteristics. These images are rendered from $24$ uniformly sampled camera viewpoints arranged on a circle centered on the garment. The circle’s radius is automatically adjusted so that, in the frontal view, the garment could nearly fill the frame. All sketches and clay renderings are output at $3840\times 2048$ resolution, and we record each camera’s transformation matrix in the standard NeRF format.
- •
A *point cloud* sampled from the garment mesh using Poisson‐disk sampling (Open3D) to capture its geometric detail. To closely mimic real-world scans or multi-view reconstructions, which emphasize the exterior surface, we adapt sampling density by occlusion: outer panels are sampled at a high density, while inner panels use a sparser density. We randomly downsample these point clouds at varying rates to improve model robustness and performance.

### 6.2. Dataset Statistics

As summarized before, GarmageSet comprises $14{,}801$ professionally modeled garments across five major categories including tops (2,888), outerwears (2,293), pants (857), skirts (1,523), dresses (6,454), and 786 garments from other categories such as sportswear, bras, pajamas and cheongsams.
Each garment is annotated along professionally defined design dimensions with over a hundred part‐wise variations (Figure  [8](https://arxiv.org/html/2504.01483v4#S5.F8)), yielding a combinatorial design space of more than $2.9454\times 10^{11}$ topologically distinct configurations.

Although smaller in size, GarmageSet covers substantially richer variation than GarmentCodeData (Korosteleva et al., 2024), which is limited to basic modifications (e.g., a single dart type for FittedShirt and single lapel style defined in SimpleLapel). To quantify structural complexity, we randomly sample $10{,}000$ garments (and $10{,}000$ panels) from each dataset and compare statistics in Figure  [11](https://arxiv.org/html/2504.01483v4#S6.F11). GarmageSet has on average $13.59_{\pm 7.89}$ panels and $46.01_{\pm 26.45}$ stitches per garment with $8.62_{\pm 5.01}$ edges per panel. By contrast, GarmentCodeData provides only $10.82_{\pm 6.29}$ panels and $30.26_{\pm 17.59}$ stitches per garment, with $6.75_{\pm 4.20}$ edges per panel, indicating significantly lower structural richness.
In terms of 3D drape fidelity, by setting the particle distance to $6$mm during simulation, GarmageSet features $72{,}560.5_{\pm 27{,}886.7}$ vertices and $141{,}064.1_{\pm 55{,}899.8}$ faces per garment; while GarmentCodeData only has $13{,}998.86_{\pm 8{,}658.42}$ vertices and $26{,}400.82_{\pm 16{,}751.82}$ faces per garment, demonstrating that GarmageSet delivers over five-fold higher mesh resolution than GarmentCodeData which indicates richer structural details.
Figure  [12](https://arxiv.org/html/2504.01483v4#S6.F12) presents representative samples from GarmageSet, visually demonstrating its high geometric fidelity and intricate structural detail. For example, complex garment foldings and shirrings (c,f); multi-layered design (d) and irregular splits (a,i,e,h) that hard to achieve with GarmentCode’s parametric formulation.

Figure: Figure 12. Representative examples from our *GarmageSet*, demonstrating the dataset’s rich diversity in garment categories, styles, and intricate folding patterns.
Refer to caption: figs/data_example.jpg

## 7. Experiments

In this section, we first detail the implementation and training protocols for GarmageNet and GarmageJigsaw then quantify our frameworks’ performance by evaluating sewing‐pattern recovery quality, 3D geometry fidelity, and sewing accuracy.

### 7.1. Implementation Details

We randomly reserved 1,024 garment assets from GarmageSet for validation, using the remaining 13,777 assets for training.

GarmageNet was trained on a single NVIDIA A100 GPU over 1–2 days using a two‐stage protocol. In the latent‐encoding stage, we trained the VAE for 200 epochs with a batch size of 256, using the AdamW optimizer at a learning rate of $5\times 10^{-4}$. This stage completes in approximately 2 hours.
In the diffusion‐generation stage, we employ a standard DDPM scheduler and train the denoiser for 20,000 epochs with a batch size of 4,096, which takes approximately 12 hours.
In conditional generation with text prompts or point clouds, we need to incorporate augmentations such as random word dropout in prompts, variable point‐cloud sampling densities, and on‐the‐fly embedding computation. Thus, extends total training time to roughly 24 hours.

We trained GarmageJigsaw using two NVIDIA RTX 4090 GPUs with a batch size of 28. The training was initialized with a learning rate of $1\times 10^{-3}$, which was gradually decreased using cosine learning rate decay, ultimately reaching $2\times 10^{-5}$ at the end of the training process. We train our GarmageJigsaw for $100$ epochs, taking approximately 27 hours in total.

### 7.2. Evaluation And Comparison

As previously demonstrated, GarmageNet can synthesize complete garment assets, encompassing 2D sewing patterns, sewing correspondences, and high‐resolution 3D initializations. Accordingly, we evaluate its generation quality across these core dimensions.

#### 7.2.1. Cross-Dataset Comparison on Sewing Pattern Quality

First, we conduct a cross-dataset study to quantify the impact of GarmageSet (real production data) on model performance and to validate GarmageNet against a representative structure-centric baseline. We compare with AIpparel (Nakayama et al., 2024), which quantizes sewing patterns into 1D token sequences and uses a large language model to decode patterns from multimodal inputs. Evaluations are performed on the synthetic GCD-MM dataset used in AIpparel and on our GarmageSet.
Because Garmage adopts a rasterized sewing-pattern representation that differs fundamentally from vector-quantized encodings, and because GarmageSet both violates several simplifying assumptions of GCD-MM (e.g., one-to-one edge-wise seams) and lacks ground-truth seam graphs and intermediate initialization cues such as per-panel translations/rotations (Section [2.1.3](https://arxiv.org/html/2504.01483v4#S2.SS1.SSS3)), AIpparel’s original metrics, such as Panel-L2, edge-wise Stitch-F1, Trans-L2, Rot-L2, are not directly applicable. We therefore introduce the following metrics that are valid on both GCD-MM and GarmageSet to ensure a fair comparison:

**Table 2. Cross-dataset comparison of GarmageNet with vector-quantization-based representative AIpparel (Nakayama et al., 2024) on GarmageSet and GCD-MM. GarmageNet generally outperforms AIpparel on per-panel intersection over union (Panel-IoU), simulation succession rate (SSR) and chamfer distance (CD), but slightly lags on predicted number of panels (#Panels) and stitch‑F1 score.**
| Exp. | Dataset | Method | Panel-IOU | #Panels | Stitch-F1 | SSR | CD (mm) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | GCD-MM | AIpparel | 0.89 | 0.85 | 0.76 | 0.59 | 59.3 |
| 2 | AIpparel-Rand | 0.43 | 0.26 | 0.17 | / | / |  |
| 3 | GarmageNet | 0.94 | 0.47 | 0.66 | 0.52 | 30.1 |  |
| 4 | GarmageSet | AIpparel | 0.53 | 0.16 | / | / | / |
| 5 | GarmageNet | 0.88 | 0.23 | 0.78 | 0.91 | 53.94 |  |

- •
Panel Shape Complexity. We use pixel-wise IoU between rasterized panels, which is able to capture internal holes, multi-edge loops, and complex contours (e.g., spirals) present in GarmageSet.
- •
Sewing Quality. GarmentCodeData encodes seams as one-to-one, full-edge correspondences, whereas GarmageSet requires many-to-many, partial-edge mappings for multilayering and pleats (enforcing 1:1 full-edge sewing can explode edge counts). In the absence of a robust sewing-quantization for GarmageSet, we report Stitch-F1 where labels are compatible—namely on GarmentCodeData and GarmageNet outputs (Exps. 1–3,5).
- •
3D Initialization Cues. As noted in Sec. [2.1](https://arxiv.org/html/2504.01483v4#S2.SS1), GarmageSet lacks per-panel rigid placements (translations/rotations), rendering AIpparel’s Trans-L2 and Rot-L2 inapplicable. We therefore evaluate drape quality using Simulation-Success Rate (SSR) and Chamfer Distance (CD) between successfully simulated predictions and ground-truth meshes.

We retrained GarmageNet on GarmentCodeData for 50 GPU-hours (RTX 4090) and AIpparel on GarmageSet for 157 GPU-hours (H20). All metrics are reported on line-art–guided generation over test splits of corresponding datasets. Table [2](https://arxiv.org/html/2504.01483v4#S7.T2) lists the detailed comparison results(^4^44Exp.1 is evaluated with AIpparel’s (Nakayama et al., 2024) officially provided checkpoints.), comparing Exp.1 vs Exp.3, Exp.4 vs Exp.5, we can conclude that GarmageNet generally outperforms AIpparel in Panel-IoU, SSR, and CD, but trails slightly in #Panels and Stitch-F1 on the GCD-MM dataset. We attribute this degradation in part to the sparsity of line-art features and the presence of occlusions, which can cause the pretrained feature extractor to miss fine segmentation details critical to #Panels or to overlook inner panels that are partially hidden (Figure [20](https://arxiv.org/html/2504.01483v4#S9.F20) (e,f)).

Figure: Figure 13. Qualitative comparison between Garmage, rigid-transformation-based and optimization-based (Liu et al., 2024d) garment initialization methods. Garmage effectively models intricate folding structures such as Jabot decorations (a), folded-over turtlenecks (b), natural knots (c), and lapel structures (d).
Refer to caption: figs/eval_ssr.png

#### 7.2.2. Sewing Accuracy Evaluation

The GarmageJigsaw module for sewing recovery consists of two main components: a point classifier and a sewing predictor. To thoroughly evaluate its effectiveness, we assess each component independently and perform an ablation study on input feature selection, comparing the fused 2D/3D features used in GarmageJigsaw with 2D-only and 3D-only variants.

The point classifier operates as a binary classifier, and we evaluate its performance using precision and recall. The classification precision (CP) measures the proportion of correctly identified sewing points (i.e., true positives) among all predicted positives, while the classification recall (CR) indicates the proportion of true positive predictions among all sewing points in the ground truth.
As shown in Table [3](https://arxiv.org/html/2504.01483v4#S7.T3), the point classifier achieves a precision of $99.16\%$ and a recall of $97.13\%$, indicating strong performance in identifying sewing points.

**Table 3. Ablation study on sewing relationship recovery, comparing the performance of GarmageJigsaw trained with both 2D and 3D features versus models trained with only 2D or 3D features. The table reports key metrics including point classification precision (CP), recall (CR), average matching distance (AMD), topological accuracy (tACC), and topological precision (tP).**
|  | CP ($\uparrow$) | CR ($\uparrow$) | AMD ($\downarrow$) | tACC ($\uparrow$) | tP ($\uparrow$) |
| --- | --- | --- | --- | --- | --- |
| GarmageJigsaw | 99.16 | 97.13 | 6.610 | 96.79 | 98.68 |
| 3D-feat Only | 99.27 | 96.99 | 7.790 | 96.36 | 97.96 |
| 2D-feat Only | 99.21 | 97.12 | 10.59 | 96.28 | 97.70 |

For the sewing predictor, we first evaluate the panel-level topological quality of the generated sewing patterns with:

- •
Accuracy (tACC): The proportion of correctly predicted sewing connections (correct sewing pairs) out of all predicted connections. Higher values indicate better topological correctness.
- •
Precision (tP): The proportion of correctly predicted sewing connections out of all predicted connections, where higher values reflect fewer false positives.

Additionally, we evaluate vertex-level sewing quality using Average Matching Distance (AMD), which calculates the average Euclidean distance between predicted sewing correspondent and ground truth correspondent for all vertices. Lower AMD values indicate better alignment between predicted and actual sewing positions.

Table [3](https://arxiv.org/html/2504.01483v4#S7.T3) summarizes the evaluation results with ablation studies on using only 2D or 3D features for sewing relationship recovery. These results confirm that combining both 3D and 2D features enables GarmageJigsaw to achieve more robust stitching recovery with lower AMD value and topological accuracy.

#### 7.2.3. Fine‐Grained 3D Initialization Evaluation

To quantify the benefits of GarmageNet’s vertex‐level initializations, we compare its simulation succession rate (SSR) against two baselines: (1) rigid transformations-based initialization as used in GarmentCodeData (Korosteleva and Sorkine-Hornung, 2023); and (2) optimization‐based initialization from raw sewing patterns (Liu et al., 2024d).

We collect 150 sewing patterns with ground truth stitching relationships from GarmageSet, recover their initial drape status with GarmentNet and drape onto our standard Size S avatar using identical simulation settings as  (Liu et al., 2024d) and compare the SSR as garments draped successfully onto the avatar without observable self-collision, body-collision, sliding errors etc.
For the rigid baseline, we leverage the per‐panel semantics in GarmageSet (Section [6.1.3](https://arxiv.org/html/2504.01483v4#S6.SS1.SSS3)) to assign each panel a fixed pose, using standard rigid placement for body, skirt, and front/back panels, and cylindrical arrangement for tubular components such as sleeves and collars.

As a result, GarmageNet achieves an SSR of 91.41%, substantially higher than rigid initialization (59.38%) and on par with optimization-based initialization (93.75%).
Figure [13](https://arxiv.org/html/2504.01483v4#S7.F13) presents representative cases, from which we can conclude that our fine-grained, per-vertex placements could provide robust draping initialization for complex draping (a), knots (c) and folding behaviors (b,d).

Figure: Figure 14. Unconditional garment generation comparison between GarmageNet, Omage (Yan et al., 2024), and Surf-D (Yu et al., 2025a). GarmageNet (left block) produces simulation-ready assets complete with vectorized sewing patterns, vertex-wise stitch relationships, and fine-grained 3D draping initializations (a,b,c,d). In contrast, Omage’s outputs (top right) exhibit incomplete panels (g), grid-like tessellation artifacts (f), erroneous stitching between non-adjacent panels (h), and spurious triangles that connect a panel’s boundary vertices back to the global origin (e). Surf-D’s meshes (bottom right) suffer from unwanted holes (i, l) and frayed, irregular boundaries (j, k). These close-up comparisons highlight GarmageNet’s superior geometric fidelity, coherent panel topology, and artifact-free mesh integrity.
Refer to caption: figs/uncond_comp.png

**Table 4. Comparison of generation quality, diversity, and efficiency between GarmageNet, Omage(Yan et al., 2024), and Surf-D(Yu et al., 2025a). Quality metrics include Minimum Matching Distance (MMD, $\times 10^{-3}m$), Jensen–Shannon Divergence (JSD), point-cloud FID (p-FID), and point-cloud KID (p-KID), where lower values indicate better fidelity. Diversity is measured by Coverage (COV, %), and efficiency is assessed based on inference GPU memory usage (Mem.) and inference speed (measured in seconds).**
| Method | Quality | Diversity | Efficiency |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| MMD ($\downarrow$) | JSD ($\downarrow$) | p-FID ($\downarrow$) | p-KID ($\downarrow$) | COV ($\uparrow$) | Mem. ($\downarrow$) | Duration ($\downarrow$) |  |
| Surf-D | 146.87 | 0.7907 | 46.61 | 0.1718 | 16.02% | 7 GB | 25.7s |
| Omages | 96.44 | 0.1185 | 29.38 | 0.1271 | 28.16% | 3.3 GB | 120s |
| Ours | 35.55 | 0.0337 | 15.34 | 0.029 | 41.02% | 4 GB | 8s |

#### 7.2.4. 3D Garment Asset Quality

We compare GarmageNet’s garment generation quality against two representative geometry-centric generation approaches: Omage (Yan et al., 2024), which typifies geometry-image-based 3D generation pipelines akin to our approach; and Surf-D (Yu et al., 2025a), an implicit-field method that generates surfaces via unsigned distance functions.
Table [4](https://arxiv.org/html/2504.01483v4#S7.T4) presents the comparison results according to five metrics:

- •
Minimum Matching Distance (MMD) measures the average closest‐distance between each real sample and its generated counterpart (units of $10^{-3}m$). A lower MMD indicates that, on average, every real garment has a very similar counterpart among the generated set.
- •
Jensen–Shannon Divergence (JSD) quantifies the overall distributional discrepancy. A lower JSD means that the probability distributions of real and generated samples are more similar.
- •
Point-cloud FID (p-FID) and KID (p-KID) assess generation fidelity using learned feature embeddings, with lower values indicating the generated feature distribution are closer to those of the real data.
- •
Coverage (COV) is the fraction of real samples matched by at least one generated sample (in percentage $\%$). A higher COV indicates broader exploration of the real data manifold, i.e., greater diversity.

For a fair comparison, all baseline methods were retrained on the full GarmageSet under the unconditional generation setting. Specifically, Omage (Yan et al., 2024) was trained at a resolution of $64\times 64$, requiring approximately 50 hours for training and consuming 3.3MB of memory with an inference time of 120 seconds per sample. Surf-D (Yu et al., 2025a) was trained at a resolution of $512$, where the VAE module took four days to train on two RTX 4090 GPUs, followed by 20 hours of diffusion model training. To compute point-cloud FID and KID scores, we adopt the pretrained PointNet++ feature extractor provided by Point-E (Nichol et al., 2022). Each method generated 128 random samples using a single NVIDIA GeForce RTX 3060 for evaluation. As reported in Table [4](https://arxiv.org/html/2504.01483v4#S7.T4), GarmageNet outperforms both Omage and Surf-D in terms of generation fidelity, diversity, and computational efficiency.

Figure [14](https://arxiv.org/html/2504.01483v4#S7.F14) presents unconditional generation results from GarmageNet alongside those of Surf-D and Omage. Omage produces a single multi-chart geometry image for the entire garment, making its outputs vulnerable to irregular UV chart packing; addressing this requires a much larger network and longer training times. As shown, Omage’s results appear coarse and often suffer from missing panels. Surf-D exemplifies a backward modeling approach, using an unsigned distance field (UDF) for generation and then extracting a triangle mesh. Consequently, it generates only a single, monolithic mesh without any explicit sewing-pattern structure, and the UDF-to-mesh conversion can introduce holes. In contrast, GarmageNet delivers panel-aware garments with complete and crisp per-panel structure, and fine-grained draping status.

Figure: Figure 15. Text conditioned garment generation results and comparison with Design2GarmentCode (Zhou et al., 2024) and Hunyuan 3D 2.5 (Zhao et al., 2025).(^†^†:)
Refer to caption: figs/cond_gen_text.jpg

Figure: Figure 16. Line-art guided garment generation results and comparison with Design2GarmentCode (Zhou et al., 2024) and Hunyuan 3D 2.5 (Zhao et al., 2025).(^†^†:)
Refer to caption: figs/cond_gen_sketch.png

## 8. Applications

We demonstrate the practical versatility and effectiveness of the proposed *GarmageNet* framework through four application scenarios that cover the full spectrum of digital garment modeling. These include interpreting abstract design concepts, automatically generating 3D garment assets from raw sewing patterns, reconstructing manufacturable sewing patterns from unstructured data, and performing conventional garment asset editing based on simple textual inputs. These scenarios showcase GarmageNet’s ability to accurately translate diverse inputs into structurally sound and visually compelling garment assets, bridging the gap between creative ideation and real-world garment production.

**Table 5. User study results for generation quality comparison of our method against state-of-the-art (SOTA) forward generation technique Design2GarmentCode (trained on GarmentCodeData), and backward generation technique Hunyuan3D 2.5. Here, Agreement evaluates the alignment between the generated garment and the design input (text or line-art sketch). Garment Aesthetic evaluates the geometric quality of the generated 3D garment asset, while Sewing Pattern Aesthetic evaluates the quality of the generated sewing pattern. We provide CLIPScore as an additional agreement evaluation for text-guided generation.**
| Method | Text-Guided Generation | Line-Art Guided Generation |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Agreement | Garment Aesthetic | Sewing Pattern Aesthetic | CLIPScore | Agreement | Garment Aesthetic | Sewing Pattern Aesthetic |  |
| GarmageNet + GarmageJigsaw | 62.50% | 85.00% | 90.42% | 0.3076 | 77.34% | 68.75% | 97.66% |
| Design2GarmentCode (Zhou et al., 2024) | 4.17% | 7.92% | 9.58% | 0.2955 | 0.0% | 10.16% | 2.34% |
| Hunyuan 3D v2.5 (Zhao et al., 2025) | 33.33% | 7.08% | 0.00% | 0.3016 | 22.66% | 21.09% | 0.0% |

### 8.1. Design Concept to Garment Generation

Generating garments directly from high-level design concepts, such as textual descriptions or minimalistic line-art sketches, significantly streamlines fashion design workflows, particularly in rapid prototyping and initial visualization stages. Unlike traditional methods that necessitate detailed technical specifications, GarmageNet interprets natural language prompts and simple sketches to automatically produce structurally correct and visually coherent 3D garments.

Qualitative evaluations supported by detailed X-ray renderings and UV-aligned normal maps reveal that GarmageNet effectively captures original design intents. The generated garments exhibit clearly defined seam structures, realistic draping, and well-articulated folds—key elements often compromised in outputs from existing frameworks such as Design2GarmentCode (forward generation) and Hunyuan3D v2.5 (backward generation).

Figure [15](https://arxiv.org/html/2504.01483v4#acmlabel4), [16](https://arxiv.org/html/2504.01483v4#acmlabel5) provide qualitative evaluations of garments generated from text prompts and line-art sketches, compared against two state-of-the-art baseline models: the forward generation approach, Design2GarmentCode(Zhou et al., 2024), trained on GarmentCodeData (Korosteleva et al., 2024), and the backward generation method, Hunyuan3D v2.5 (Zhao et al., 2025), trained on massive 3D assets.
For each generated garment, we present X-ray renderings to reveal the underlying geometric structures and UV-aligned normal maps to intuitively assess the quality of the generated sewing patterns and the detailed fold structures. Our outputs demonstrate clear and accurate seam structures, precise garment draping, and refined folds which are inadequately represented by the baseline methods.

Leveraging the line-art sketch-conditioned GarmageNet as a baseline, our framework could further enable in-the-wild image guided garment generation. Specifically, we employ a LoRA (Hu et al., 2022) fine-tuned FLUX (Labs, 2024) model to transfer real-world photographs and design sketches to GarmageSet style line-art sketches. These sketches subsequently guide the Garmage generation process, with results illustrated in Figure [16](https://arxiv.org/html/2504.01483v4#acmlabel5) (e-h), underscoring the model’s enhanced versatility and real-world applicability.

A comprehensive user study involving 20 professional fashion designers, pattern makers, and 3D apparel modelers validated our findings quantitatively. Each participant is asked to review 48 garment generation results (24 text prompts and 24 sketches randomly sampled from $1000+$) and select which method’s result was best under three criteria:

- •
Agreement with the input prompt (i.e. how well the 3D garment matches the described or drawn design);
- •
Garment Aesthetic (overall visual and geometric quality of the 3D garment model);
- •
Sewing Pattern Aesthetic (quality and plausibility of the underlying pattern structure, as evident in the model and its UV seams).

The aggregated preference results (normalized percentages of selections for each model) in Table [5](https://arxiv.org/html/2504.01483v4#S8.T5) indicate GarmageNet significantly outperformed the baselines across all metrics, being preferred in over 60% of cases for Agreement, 85% for Garment Aesthetic, and approximately 90% for Sewing Pattern Aesthetic in text-guided generation; and 77% for Agreement. 68.75%for Garment Aesthetic and 97.66% for Sewing Pattern Aesthetic. Further, GarmageNet achieved the highest normalized CLIPScore (0.3076), confirming superior semantic alignment with text descriptions.

Figure: Figure 17. Automatic garment modeling from raw sewing patterns. Given flat sewing patterns without sewing relationship, For clarity, we highlight several panels in the raw sewing pattern and label their corresponding generated Garmages with 3D point visualization.
Refer to caption: figs/garmage-result_mask_6.png

### 8.2. Automatic Garment Modeling

Beyond its central role in gaming, virtual reality, and digital fashion, automatic garment modeling is equally critical for apparel manufacturing. By enabling manufacturers to visualize and validate sewing patterns before physical production, it helps reduce sampling costs, accelerate iteration, and minimize material waste. However, 3D garment modeling is traditionally a highly skill-demanding process, and the majority of industrial or internet-available resources contain only raw 2D sewing patterns without corresponding 3D assets. This gap underscores the necessity of an automatic pipeline capable of converting 2D patterns into faithful, simulation-ready 3D garments.
GarmageNet directly addresses this challenge by leveraging the masked latent encoding (Section [4.2.1](https://arxiv.org/html/2504.01483v4#S4.SS2.SSS1)) to provide fine-grained 3D initialization via the Garmage representation and establishes vertex-level stitching through GarmageJigsaw.

Figure [17](https://arxiv.org/html/2504.01483v4#S8.F17) demonstrates garment generation results from raw sewing patterns. Several original panels are shown alongside their generated Garmages, highlighting that GarmageNet robustly handles automatic garment modeling even with unconventional designs such as hem panels (Figure [17](https://arxiv.org/html/2504.01483v4#S8.F17) (a,f)). Moreover, although no explicit symmetry constraints were imposed during training, GarmageNet successfully captures symmetry cues inherent in the data (Figure [17](https://arxiv.org/html/2504.01483v4#S8.F17) (d)), producing garments with strong bilateral consistency, particularly across sleeve panels.

Figure: Figure 18. Point‐cloud‐conditioned garment synthesis with GarmageNet. Each row (a–f) shows: (left) an unstructured, sparse point cloud captured from a draped garment; (center) the generated Garmage representation—consisting of per‐panel geometry images (colored) and inferred panel contours (outlined); and (right) the final simulation‐ready 3D garment asset, obtained by vectorizing the extracted sewing patterns, recovering vertex‐wise stitches, and applying physics‐based draping. These results demonstrate GarmageNet’s ability to transform noisy, incomplete point clouds into fully structured sewing patterns and high‐fidelity draped garments. We note, however, that the network may be leveraging the non‐uniform sampling density of the input point clouds—implicitly revealing panel structure—to achieve these reconstructions.
Refer to caption: figs/cond_gen_point.png

### 8.3. Sewing Pattern Recovery

Advancements in 3D scanning and multi-view reconstruction technologies have greatly facilitated capturing realistic garment shapes, typically represented as unstructured point clouds. However, such raw 3D data lacks the structured information essential for garment production, thus necessitating effective methods for recovering structured sewing patterns from unstructured 3D representations.

GarmageNet addresses this critical industry challenge by accurately transforming point-cloud data of draped garments into structured Garmages, successfully recovering detailed sewing patterns. Figure  [18](https://arxiv.org/html/2504.01483v4#S8.F18) showcases recovered sewing patterns, highlighting intricate folds and precise seam alignments. These recovered patterns closely match their original counterparts, demonstrating high accuracy in panel shapes, seam definitions, and adherence to industry production standards.

Qualitative analysis indicates that GarmageNet robustly identifies precise panel boundaries, seam connections, and garment folds from noisy input data, achieving reliable sewing pattern recovery even in complex garment configurations. This functionality positions GarmageNet uniquely within digital garment pipelines, effectively linking unstructured scan data to structured, production-ready garment assets, thereby significantly enhancing practical applicability in apparel manufacturing workflows.

Figure: Figure 19. Interactive garment editing using conventional design instructions. Starting from an initial Garmage (top-left), users issue sequential edits—e.g. replacing a round neckline with a shirt collar, adding a fitted waist, and switching to a standing collar—while all unchanged panels remain in grey and only the edited panels (in color) are updated in their geometry images. Each intermediate Garmage is decoded into a full garment asset and re-simulated, demonstrating how our framework seamlessly incorporates standard pattern-making edits into the generation and draping pipeline.
Refer to caption: figs/cond_editing.png

### 8.4. Progressive Generation and Editing

Beyond generating garments directly from text prompts, our framework also supports advanced garment editing functionalities, such as adding, deleting, or replacing components of an existing garment. This capability significantly enhances the flexibility of the design process, allowing designers to iteratively refine garments based on new inputs while preserving key structural features.

Recall from Eq. [1](https://arxiv.org/html/2504.01483v4#S4.E1) that a Garmage consists of a set of panels, each represented by a 2D dimension $D_{i}$, a 3D axis-aligned bounding box $B_{i}$, and a normalized geometry image patch $I_{i}$. After generating a garment using text prompts, users can modify the original prompts to reflect desired changes, such as removing or replacing specific garment components.

When a text prompt is updated, the garment is regenerated, and the newly generated panels are compared against the original panels based on their 2D dimensions and 3D bounding boxes. Panels with high similarity are marked for retention, while those with low similarity are flagged for modification.
The editing process is akin to inpainting in image generation models, where only the panels requiring modification are regenerated. Retained panels are treated in a way similar to diffusion-based denoising, where the original features are preserved and augmented with noise according to the current timestep, guiding the model to retain the established characteristics of those panels. In this way, the modifications are localized to the relevant areas of the garment without disrupting the overall design, and new panels are generated in a manner that ensures smooth transitions at the interfaces between modified and retained panels (e.g., sleeve holes), maintaining coherence in both geometry and design.

Figure [19](https://arxiv.org/html/2504.01483v4#S8.F19) illustrates an progressive generation process where we first generate a fitted, sleeveless dress from text prompts (a), then add long sleeves to the dress (b), and modify the sleeves to puff sleeves (c). Next, we add standing lapel collars to the dress (d) and modify them to shirt collars (e). Finally, if the user is dissatisfied with the generated result, we can even modify the entire initial bodice panels (f). Note that the structural integrity and stylistic consistency are preserved during the whole editing process.

## 9. Conclusion

In this work, we introduced *GarmageNet*, the first end-to-end framework for unified 2D–3D garment synthesis. At its core is *Garmage*, a novel panel-aligned geometry-image representation that encodes both discrete sewing-pattern structure and continuous draping geometry into a compact, image-based format. By training a latent-diffusion transformer on Garmage tokens, our approach enables both unconditional and conditional generation from diverse design modalities—including text, sketches, point clouds, and raw sewing patterns—while preserving fine-grained panel topology and delivering high-fidelity, simulation-ready initializations. We further presented *GarmageJigsaw*, a dedicated module that leverages 2D silhouettes and 3D spatial cues to recover vertex-level sewing relationships, seamlessly converting generated Garmages into vectorized sewing patterns and triangulated meshes compatible with physics-based simulation.

To support this framework, we introduced *GarmageSet*, the largest garment dataset to date built from real production data. It contains 14,801 professionally drafted sewing patterns paired with fine-grained 3D assets, detailed structural and style annotations, and multi-view renderings with point clouds. By bridging industrial-grade data quality with rich multi-modal supervision, GarmageSet provides a robust foundation for advancing garment modeling research toward deployable manufacturing pipelines.

Comprehensive evaluations demonstrate that GarmageNet outperforms state-of-the-art structure-centric and geometry-centric approaches in terms of quality, diversity, and robustness, and achieves higher simulation success rates compared to rigid and optimization-based automatic initialization methods.

Figure: Figure 20. Failure cases of our proposed framework. (a–b) Missing internal seams: while interior stitch lines and pocket attachments are present in training, GarmageJigsaw receives only boundary points at inference, preventing correct stitching of pocket-like panels. (c–d) Pose sensitivity: since GarmageSet contains only garments draped on a size-S A-pose avatar, point-cloud–conditioned generation can fail under pose variations. (e–f) Missing small or auxiliary parts: our zero-padding scheme may cause the loss of small panels such as cuffs or thin stripes.
Refer to caption: figs/limitations.png

## 10. Limitations and Future Work

While GarmageNet demonstrates strong performance in unified garment modeling, several limitations remain:

##### Lacking body shape and pose variance

To control dataset preparation costs, GarmageSet currently contains garments drafted only for a standard size-S avatar posed in A-pose. As a result, GarmageNet may be sensitive to pose or shape variations, particularly under point-cloud–conditioned generation (Figure [20](https://arxiv.org/html/2504.01483v4#S9.F20) (c,d)). Incorporating body-shape and pose diversity is a priority for future releases.

##### Handling of interior seams and pocket-like components

Although GarmageNet and GarmageJigsaw are trained with interior seams and panels that attach along them (for instance, pocket panels in Figure [20](https://arxiv.org/html/2504.01483v4#S9.F20) (a,b)). During inference, we only pass boundary facets to GarmageJigsaw to identify sewing relationships between those boundary facets. This limitation prevents the model from stitching the generated pocket panels to bodice panels due to the absence of interior edges on bodice panels (Figure [20](https://arxiv.org/html/2504.01483v4#S9.F20) (b)).

##### Multi-layer designs and small-panel handling

Because GarmageSet includes garments with layered collars and sleeve designs, GarmageNet occasionally generates redundant sleeve panels, leading to reduced accuracy in panel-count metrics (#Panels) as shown in Table [2](https://arxiv.org/html/2504.01483v4#S7.T2). Moreover, the zero-padding strategy adopted during training can cause small or thin panels (e.g., cuffs or narrow stripes) to be omitted (Figure [20](https://arxiv.org/html/2504.01483v4#S9.F20) (c)).

##### Future Work

We plan to expand GarmageSet to include a broader spectrum of body shapes and poses beyond the current size-S A-pose, thereby reducing sensitivity to pose and shape variations in conditional generation. We also intend to enhance seam representations by adding interior seam annotations and pocket-attachment edges, enabling GarmageJigsaw to accurately handle pocket-like components. Furthermore, we will continue to refine the Garmage representation and optimize the architectures of GarmageNet and GarmageJigsaw to better accommodate multi-layered designs and small panels.

###### Acknowledgements.

###### Acknowledgements.

## References

- (1)
- Berthouzoz et al. (2013)
Floraine Berthouzoz, Akash Garg, Danny M Kaufman, Eitan Grinspun, and Maneesh Agrawala. 2013.
Parsing sewing patterns into 3D garments.
*Acm Transactions on Graphics (TOG)* 32, 4 (2013), 1–12.
- Bertiche et al. (2020)
Hugo Bertiche, Meysam Madadi, and Sergio Escalera. 2020.
Cloth3d: clothed 3d humans. In *European Conference on Computer Vision*. Springer, 344–359.
- Bhatnagar et al. (2019)
Bharat Lal Bhatnagar, Garvita Tiwari, Christian Theobalt, and Gerard Pons-Moll. 2019.
Multi-garment net: Learning to dress 3d people from images. In *Proceedings of the IEEE/CVF international conference on computer vision*. 5420–5430.
- Bian et al. (2024)
Siyuan Bian, Chenghao Xu, Yuliang Xiu, Artur Grigorev, Zhen Liu, Cewu Lu, Michael J Black, and Yao Feng. 2024.
ChatGarment: Garment Estimation, Generation and Editing via Large Language Models.
*arXiv preprint arXiv:2412.17811* (2024).
- Black et al. (2023)
Michael J. Black, Priyanka Patel, Joachim Tesch, and Jinlong Yang. 2023.
BEDLAM: A Synthetic Dataset of Bodies Exhibiting Detailed Lifelike Animated Motion. In *Proceedings IEEE/CVF Conf. on Computer Vision and Pattern Recognition (CVPR)*. 8726–8737.
- Chen et al. (2024)
Honghu Chen, Yuxin Yao, and Juyong Zhang. 2024.
Neural-ABC: neural parametric models for articulated body with clothes.
*IEEE Transactions on Visualization and Computer Graphics* 31, 2 (2024), 1478–1495.
- Chen et al. (2022)
Xipeng Chen, Guangrun Wang, Dizhong Zhu, Xiaodan Liang, Philip Torr, and Liang Lin. 2022.
Structure-preserving 3d garment modeling with neural sewing machines.
*Advances in Neural Information Processing Systems* 35 (2022), 15147–15159.
- Cuturi (2013)
Marco Cuturi. 2013.
Sinkhorn distances: Lightspeed computation of optimal transport.
*Advances in neural information processing systems* 26 (2013).
- Fashionary (2016)
Fashionary. 2016.
*Fashionpedia: The Visual Dictionary of Fashion Design*.
Fashionary International Limited.
- Feng et al. (2022)
Xudong Feng, Wenchao Huang, Weiwei Xu, and Huamin Wang. 2022.
Learning-Based Bending Stiffness Parameter Estimation by a Drape Tester.
*ACM Trans. Graph.* 41, 6, Article 221 (Nov. 2022), 16 pages.
- Fischler and Bolles (1981)
Martin A Fischler and Robert C Bolles. 1981.
Random sample consensus: a paradigm for model fitting with applications to image analysis and automated cartography.
*Commun. ACM* 24, 6 (1981), 381–395.
- Gu et al. (2002)
Xianfeng Gu, Steven J Gortler, and Hugues Hoppe. 2002.
Geometry images. In *Proceedings of the 29th annual conference on Computer graphics and interactive techniques*. 355–361.
- Gundogdu et al. (2019)
Erhan Gundogdu, Victor Constantin, Amrollah Seifoddini, Minh Dang, Mathieu Salzmann, and Pascal Fua. 2019.
Garnet: A two-stream network for fast and accurate 3d cloth draping. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 8739–8748.
- Guo et al. (2025c)
Dewen Guo, Zhendong Wang, Zegao Liu, Sheng Li, Guoping Wang, Yin Yang, and Huamin Wang. 2025c.
Fast Physics-Based Modeling of Knots and Ties using Templates. In *Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers* *(SIGGRAPH Conference Papers ’25)*. Association for Computing Machinery, New York, NY, USA, Article 43, 9 pages.
- Guo et al. (2025d)
Dewen Guo, Zhendong Wang, Zegao Liu, Sheng Li, Guoping Wang, Yin Yang, and Huamin Wang. 2025d.
Progressive Outfit Assembly and Instantaneous Pose Transfer. In *SIGGRAPH Asia 2025 Conference Papers* *(SA Conference Papers ’25)*. Association for Computing Machinery, New York, NY, USA.
- Guo et al. (2022)
Haoxiang Guo, Shilin Liu, Hao Pan, Yang Liu, Xin Tong, and Baining Guo. 2022.
Complexgen: Cad reconstruction by b-rep chain complex generation.
*ACM Transactions on Graphics (TOG)* 41, 4 (2022), 1–18.
- Guo et al. (2025a)
Jingfeng Guo, Jinnan Chen, Weikai Chen, Zhenyu Sun, Lanjiong Li, Baozhu Zhao, Lingting Zhu, Xin Wang, and Qi Liu. 2025a.
GarmentX: Autoregressive Parametric Representations for High-Fidelity 3D Garment Generation.
*arXiv preprint arXiv:2504.20409* (2025).
- Guo et al. (2025b)
Mengqi Guo, Chen Li, Yuyang Zhao, and Gim Hee Lee. 2025b.
TreeSBA: Tree-Transformer for Self-Supervised Sequential Brick Assembly. In *European Conference on Computer Vision*. Springer, 35–51.
- He et al. (2025)
Chengzhu He, Zhendong Wang, Zhaorui Meng, Junfeng Yao, Shihui Guo, and Huamin Wang. 2025.
Automated Task Scheduling for Cloth and Deformable Body Simulations in Heterogeneous Computing Environments. In *Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers* *(SIGGRAPH Conference Papers ’25)*. Association for Computing Machinery, New York, NY, USA, Article 24, 11 pages.
- He et al. (2024)
Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, and Lan Xu. 2024.
DressCode: Autoregressively Sewing and Generating Garments from Text Guidance.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–13.
- Hu et al. (2022)
Edward J Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, Weizhu Chen, et al. 2022.
Lora: Low-rank adaptation of large language models.
*ICLR* 1, 2 (2022), 3.
- Jayaraman et al. (2022)
Pradeep Kumar Jayaraman, Joseph G Lambourne, Nishkrit Desai, Karl DD Willis, Aditya Sanghi, and Nigel JW Morris. 2022.
Solidgen: An autoregressive model for direct b-rep synthesis.
*arXiv preprint arXiv:2203.13944* (2022).
- Jiang et al. (2020)
Boyi Jiang, Juyong Zhang, Yang Hong, Jinhao Luo, Ligang Liu, and Hujun Bao. 2020.
Bcnet: Learning body and cloth shape from a single image. In *Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28, 2020, Proceedings, Part XX 16*. Springer, 18–35.
- Korosteleva et al. (2024)
Maria Korosteleva, Timur Levent Kesdogan, Fabian Kemper, Stephan Wenninger, Jasmin Koller, Yuhan Zhang, Mario Botsch, and Olga Sorkine-Hornung. 2024.
GarmentCodeData: A Dataset of 3D Made-to-Measure Garments With Sewing Patterns. In *Computer Vision – ECCV 2024*.
- Korosteleva and Lee (2021)
Maria Korosteleva and Sung-Hee Lee. 2021.
Generating datasets of 3d garments with sewing patterns.
*arXiv preprint arXiv:2109.05633* (2021).
- Korosteleva and Lee (2022)
Maria Korosteleva and Sung-Hee Lee. 2022.
NeuralTailor: Reconstructing Sewing Pattern Structures from 3D Point Clouds of Garments.
*ACM Trans. Graph.* 41, 4 (2022), 16 pages.
[doi:10.1145/3528223.3530179](https://doi.org/10.1145/3528223.3530179)
- Korosteleva and Sorkine-Hornung (2023)
Maria Korosteleva and Olga Sorkine-Hornung. 2023.
GarmentCode: Programming Parametric Sewing Patterns.
*ACM Transaction on Graphics* 42, 6 (2023), 16 pages.
[doi:10.1145/3618351](https://doi.org/10.1145/3618351)
SIGGRAPH ASIA 2023 issue.
- Labs (2024)
Black Forest Labs. 2024.
FLUX.
[https://github.com/black-forest-labs/flux](https://github.com/black-forest-labs/flux).
- Li et al. (2024e)
Lei Li, Songyou Peng, Zehao Yu, Shaohui Liu, Rémi Pautrat, Xiaochuan Yin, and Marc Pollefeys. 2024e.
3D Neural Edge Reconstruction. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 21219–21229.
- Li et al. (2025a)
Ren Li, Cong Cao, Corentin Dumery, Yingxuan You, Hao Li, and Pascal Fua. 2025a.
Single View Garment Reconstruction Using Diffusion Mapping Via Pattern Coordinates. In *Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers*. 1–11.
- Li et al. (2024b)
Ren Li, Corentin Dumery, Zhantao Deng, and Pascal Fua. 2024b.
Reconstruction of manipulated garment with guided deformation prior.
*Advances in Neural Information Processing Systems* 37 (2024), 58637–58662.
- Li et al. (2024c)
Ren Li, Corentin Dumery, Benoît Guillard, and Pascal Fua. 2024c.
Garment Recovery with Shape and Deformation Priors. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 1586–1595.
- Li et al. (2024d)
Ren Li, Benoît Guillard, and Pascal Fua. 2024d.
Isp: Multi-layered garment draping with implicit sewing patterns.
*Advances in Neural Information Processing Systems* 36 (2024).
- Li et al. (2025b)
Xinyu Li, Qi Yao, and Yuanda Wang. 2025b.
GarmentDiffusion: 3D Garment Sewing Pattern Generation with Multimodal Diffusion Transformers.
arXiv:2504.21476 [cs.CV]
[https://arxiv.org/abs/2504.21476](https://arxiv.org/abs/2504.21476)
- Li et al. (2025c)
Xuan Li, Chang Yu, Wenxin Du, Ying Jiang, Tianyi Xie, Yunuo Chen, Yin Yang, and Chenfanfu Jiang. 2025c.
Dress-1-to-3: Single Image to Simulation-Ready 3D Outfit with Diffusion Prior and Differentiable Physics.
*ACM Transactions on Graphics (TOG)* 44, 4 (2025), 1–16.
- Li et al. (2024a)
Yifei Li, Hsiao-yu Chen, Egor Larionov, Nikolaos Sarafianos, Wojciech Matusik, and Tuur Stuyck. 2024a.
Diffavatar: Simulation-ready garment optimization with differentiable simulation. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 4368–4378.
- Lin et al. (2023)
Siyou Lin, Boyao Zhou, Zerong Zheng, Hongwen Zhang, and Yebin Liu. 2023.
Leveraging intrinsic properties for non-rigid garment alignment. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 14485–14496.
- Liu et al. (2024d)
Chen Liu, Weiwei Xu, Yin Yang, and Huamin Wang. 2024d.
Automatic Digital Garment Initialization from Sewing Patterns.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–12.
- Liu et al. (2023)
Lijuan Liu, Xiangyu Xu, Zhijie Lin, Jiabin Liang, and Shuicheng Yan. 2023.
Towards Garment Sewing Pattern Reconstruction from a Single Image.
*ACM Transactions on Graphics (SIGGRAPH Asia)* (2023).
- Liu et al. (2024a)
Shengqi Liu, Yuhao Cheng, Zhuo Chen, Xingyu Ren, Wenhan Zhu, Lincheng Li, Mengxiao Bi, Xiaokang Yang, and Yichao Yan. 2024a.
Multimodal Latent Diffusion Model for Complex Sewing Pattern Generation.
arXiv:2412.14453 [cs.CV]
[https://arxiv.org/abs/2412.14453](https://arxiv.org/abs/2412.14453)
- Liu et al. (2024c)
Yufei Liu, Junshu Tang, Chu Zheng, Shijie Zhang, Jinkun Hao, Junwei Zhu, and Dongjin Huang. 2024c.
ClotheDreamer: Text-Guided Garment Generation with 3D Gaussians.
arXiv:2406.16815 [cs.CV]
- Liu et al. (2024b)
Zhen Liu, Yao Feng, Yuliang Xiu, Weiyang Liu, Liam Paull, Michael J Black, and Bernhard Schölkopf. 2024b.
Ghost on the Shell: An Expressive Representation of General 3D Shapes. In *ICLR*.
- Loper et al. (2015)
Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, and Michael J. Black. 2015.
SMPL: A Skinned Multi-Person Linear Model.
*ACM Trans. Graphics (Proc. SIGGRAPH Asia)* 34, 6 (Oct. 2015), 248:1–248:16.
- Lu et al. (2024)
Jiaxin Lu, Yifan Sun, and Qixing Huang. 2024.
Jigsaw: Learning to assemble multiple fractured objects.
*Advances in Neural Information Processing Systems* 36 (2024).
- Luo et al. (2024)
Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, and Xiaoguang Han. 2024.
GarVerseLOD: High-Fidelity 3D Garment Reconstruction from a Single In-the-Wild Image using a Dataset with Levels of Details.
*ACM Transactions on Graphics (TOG)* (2024).
- Ma et al. (2020)
Qianli Ma, Jinlong Yang, Anurag Ranjan, Sergi Pujades, Gerard Pons-Moll, Siyu Tang, and Michael J Black. 2020.
Learning to dress 3d people in generative clothing. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 6469–6478.
- Macklin (2022)
Miles Macklin. 2022.
Warp: A High-performance Python Framework for GPU Simulation and Graphics.
[https://github.com/nvidia/warp](https://github.com/nvidia/warp).
NVIDIA GPU Technology Conference (GTC).
- Mo et al. (2019)
Kaichun Mo, Paul Guerrero, Li Yi, Hao Su, Peter Wonka, Niloy Mitra, and Leonidas J Guibas. 2019.
Structurenet: Hierarchical graph networks for 3d shape generation.
*arXiv preprint arXiv:1908.00575* (2019).
- Mo et al. (2020)
Kaichun Mo, Paul Guerrero, Li Yi, Hao Su, Peter Wonka, Niloy J Mitra, and Leonidas J Guibas. 2020.
StructEdit: Learning structural shape variations. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 8859–8868.
- Nakayama et al. (2024)
Kiyohiro Nakayama, Jan Ackermann, Timur Levent Kesdogan, Yang Zheng, Maria Korosteleva, Olga Sorkine-Hornung, Leonidas Guibas, Guandao Yang, and Gordon Wetzstein. 2024.
AIpparel: A Large Multimodal Generative Model for Digital Garments.
*Arxiv* (2024).
- Narain et al. (2012)
Rahul Narain, Armin Samii, and James F O’brien. 2012.
Adaptive anisotropic remeshing for cloth simulation.
*ACM transactions on graphics (TOG)* 31, 6 (2012), 1–10.
- Nichol et al. (2022)
Alex Nichol, Heewoo Jun, Prafulla Dhariwal, Pamela Mishkin, and Mark Chen. 2022.
Point-e: A system for generating 3d point clouds from complex prompts.
*arXiv preprint arXiv:2212.08751* (2022).
- Patel et al. (2020)
Chaitanya Patel, Zhouyingcheng Liao, and Gerard Pons-Moll. 2020.
Tailornet: Predicting clothing in 3d as a function of human pose, shape and garment style. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*. 7365–7375.
- Pons-Moll et al. (2017)
Gerard Pons-Moll, Sergi Pujades, Sonny Hu, and Michael J Black. 2017.
ClothCap: Seamless 4D clothing capture and retargeting.
*ACM Transactions on Graphics (ToG)* 36, 4 (2017), 1–15.
- Qiu et al. (2023)
Lingteng Qiu, Guanying Chen, Jiapeng Zhou, Mutian Xu, Junle Wang, and Xiaoguang Han. 2023.
Rec-mv: Reconstructing 3d dynamic cloth from monocular videos. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 4637–4646.
- Rognant et al. (1999)
L Rognant, Jean-Marc Chassery, S Goze, and JG Planes. 1999.
The Delaunay constrained triangulation: the Delaunay stable algorithms. In *1999 IEEE International Conference on Information Visualization (Cat. No. PR00210)*. IEEE, 147–152.
- Rong et al. (2024)
Boxiang Rong, Artur Grigorev, Wenbo Wang, Michael J. Black, Bernhard Thomaszewski, Christina Tsalicoglou, and Otmar Hilliges. 2024.
Gaussian Garments: Reconstructing Simulation-Ready Clothing with Photorealistic Appearance from Multi-View Video.
arXiv:2409.08189 [cs.CV]
- Sander et al. (2003)
Pedro V Sander, Zoë J Wood, Steven Gortler, John Snyder, and Hugues Hoppe. 2003.
Multi-chart geometry images.
(2003).
- Santesteban et al. (2019)
Igor Santesteban, Miguel A Otaduy, and Dan Casas. 2019.
Learning-based animation of clothing for virtual try-on. In *Computer Graphics Forum*, Vol. 38. Wiley Online Library, 355–366.
- Sarafianos et al. (2024)
Nikolaos Sarafianos, Tuur Stuyck, Xiaoyu Xiang, Yilei Li, Jovan Popovic, and Rakesh Ranjan. 2024.
Garment3dgen: 3d garment stylization and texture generation.
*arXiv preprint arXiv:2403.18816* (2024).
- Srinivasan et al. (2025)
Pratul P Srinivasan, Stephan J Garbin, Dor Verbin, Jonathan T Barron, and Ben Mildenhall. 2025.
Nuvo: Neural uv mapping for unruly 3d representations. In *European Conference on Computer Vision*. Springer, 18–34.
- Tang et al. (2014)
Min Tang, Ruofeng Tong, Zhendong Wang, and Dinesh Manocha. 2014.
Fast and Exact Continuous Collision Detection with Bernstein Sign Classification.
33, 6, Article 186 (Nov. 2014), 8 pages.
- Tatsukawa et al. (2025)
Yuki Tatsukawa, Anran Qi, I-Chao Shen, and Takeo Igarashi. 2025.
GarmentImage: Raster Encoding of Garment Sewing Patterns with Diverse Topologies. In *Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers*. 1–11.
- Tiwari et al. (2020)
Garvita Tiwari, Bharat Lal Bhatnagar, Tony Tung, and Gerard Pons-Moll. 2020.
Sizer: A dataset and model for parsing 3d clothing and learning size sensitive 3d clothing. In *Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28, 2020, Proceedings, Part III 16*. Springer, 1–18.
- Tkachenko et al. (2025)
Maxim Tkachenko, Mikhail Malyuk, Andrey Holmanyuk, and Nikolai Liubimov. 2020-2025.
Label Studio: Data labeling software.
[https://github.com/HumanSignal/label-studio](https://github.com/HumanSignal/label-studio)
Open source software available from https://github.com/HumanSignal/label-studio.
- Tochilkin et al. (2024)
Dmitry Tochilkin, David Pankratz, Zexiang Liu, Zixuan Huang, Adam Letts, Yangguang Li, Ding Liang, Christian Laforte, Varun Jampani, and Yan-Pei Cao. 2024.
Triposr: Fast 3d object reconstruction from a single image.
*arXiv preprint arXiv:2403.02151* (2024).
- Wang et al. (2018a)
Tongtong Wang, Min Tang, Zhendong Wang, and Ruofeng Tong. 2018a.
Accurate Self-collision Detection using Enhanced Dual-cone Method.
*Computers & Graphics* 73 (2018), 70–79.
- Wang et al. (2024b)
Wenbo Wang, Hsuan-I Ho, Chen Guo, Boxiang Rong, Artur Grigorev, Jie Song, Juan Jose Zarate, and Otmar Hilliges. 2024b.
4D-DRESS: A 4D Dataset of Real-World Human Clothing With Semantic Annotations. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 550–560.
- Wang et al. (2024a)
Zhengqing Wang, Jiacheng Chen, and Yasutaka Furukawa. 2024a.
PuzzleFusion++: Auto-agglomerative 3D Fracture Assembly by Denoise and Verify.
*arXiv preprint arXiv:2406.00259* (2024).
- Wang et al. (2015)
Zhendong Wang, Min Tang, Ruofeng Tong, and Dinesh Manocha. 2015.
TightCCD: Efficient and robust continuous collision detection using tight error bounds.
*Computer Graphics Forum* 34, 7 (2015), 289–298.
- Wang et al. (2016)
Zhendong Wang, Tongtong Wang, Min Tang, and Ruofeng Tong. 2016.
Efficient and robust strain limiting and treatment of simultaneous collisions with semidefinite programming.
*Computational Visual Media* 2, 2 (2016), 119–130.
- Wang et al. (2018b)
Zhendong Wang, Longhua Wu, Marco Fratarcangeli, Min Tang, and Huamin Wang. 2018b.
Parallel multigrid for nonlinear cloth simulation.
*Computer Graphics Forum* 37, 7 (2018), 131–141.
- Wang et al. (2023)
Zhendong Wang, Yin Yang, and Huamin Wang. 2023.
Stable Discrete Bending by Analytic Eigensystem and Adaptive Orthotropic Geometric Stiffness.
*ACM Trans. Graph.* 42, 6, Article 175 (Dec. 2023), 16 pages.
- Wu et al. (2022)
Botao Wu, Zhendong Wang, and Huamin Wang. 2022.
A GPU-based multilevel additive schwarz preconditioner for cloth and deformable body simulation.
*ACM Trans. Graph.* 41, 4, Article 63 (July 2022), 14 pages.
- Wu et al. (2024)
Xiaoyang Wu, Li Jiang, Peng-Shuai Wang, Zhijian Liu, Xihui Liu, Yu Qiao, Wanli Ouyang, Tong He, and Hengshuang Zhao. 2024.
Point Transformer V3: Simpler, Faster, Stronger. In *CVPR*.
- Xiang et al. (2020)
Donglai Xiang, Fabian Prada, Chenglei Wu, and Jessica Hodgins. 2020.
Monoclothcap: Towards temporally coherent clothing capture from monocular rgb video. In *2020 International Conference on 3D Vision (3DV)*. IEEE, 322–332.
- Xiang et al. (2024)
Jianfeng Xiang, Zelong Lv, Sicheng Xu, Yu Deng, Ruicheng Wang, Bowen Zhang, Dong Chen, Xin Tong, and Jiaolong Yang. 2024.
Structured 3d latents for scalable and versatile 3d generation.
*arXiv preprint arXiv:2412.01506* (2024).
- Xie et al. (2024)
Tianyi Xie, Zeshun Zong, Yuxing Qiu, Xuan Li, Yutao Feng, Yin Yang, and Chenfanfu Jiang. 2024.
Physgaussian: Physics-integrated 3d gaussians for generative dynamics. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 4389–4398.
- Xiu et al. (2023)
Yuliang Xiu, Jinlong Yang, Xu Cao, Dimitrios Tzionas, and Michael J. Black. 2023.
ECON: Explicit Clothed humans Optimized via Normal integration. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*.
- Xu et al. (2024)
Xiang Xu, Joseph Lambourne, Pradeep Jayaraman, Zhengqing Wang, Karl Willis, and Yasutaka Furukawa. 2024.
Brepgen: A b-rep generative diffusion model with structured latent geometry.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–14.
- Yan et al. (2024)
Xingguang Yan, Han-Hung Lee, Ziyu Wan, and Angel X Chang. 2024.
An object is worth 64x64 pixels: Generating 3d object via image diffusion.
*arXiv preprint arXiv:2408.03178* (2024).
- Yang et al. (2025)
An Yang, Anfeng Li, Baosong Yang, Beichen Zhang, Binyuan Hui, Bo Zheng, Bowen Yu, Chang Gao, Chengen Huang, Chenxu Lv, Chujie Zheng, Dayiheng Liu, Fan Zhou, Fei Huang, Feng Hu, Hao Ge, Haoran Wei, Huan Lin, Jialong Tang, Jian Yang, Jianhong Tu, Jianwei Zhang, Jianxin Yang, Jiaxi Yang, Jing Zhou, Jingren Zhou, Junyang Lin, Kai Dang, Keqin Bao, Kexin Yang, Le Yu, Lianghao Deng, Mei Li, Mingfeng
Xue, Mingze Li, Pei Zhang, Peng Wang, Qin Zhu, Rui Men, Ruize Gao, Shixuan Liu, Shuang Luo, Tianhao Li, Tianyi Tang, Wenbiao Yin, Xingzhang Ren, Xinyu Wang, Xinyu Zhang, Xuancheng Ren, Yang Fan, Yang Su, Yichang Zhang, Yinger Zhang, Yu Wan, Yuqiong Liu, Zekun Wang, Zeyu Cui, Zhenru Zhang, Zhipeng Zhou, and Zihan Qiu. 2025.
Qwen3 Technical Report.
*arXiv preprint arXiv:2505.09388* (2025).
- Yu et al. (2025b)
Fenggen Yu, Yiming Qian, Xu Zhang, Francisca Gil-Ureta, Brian Jackson, Eric Bennett, and Hao Zhang. 2025b.
Dpa-net: Structured 3d abstraction from sparse views via differentiable primitive assembly. In *European Conference on Computer Vision*. Springer, 454–471.
- Yu and Wang (2024)
Jiawang Yu and Zhendong Wang. 2024.
Super-Resolution Cloth Animation with Spatial and Temporal Coherence.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–14.
- Yu et al. (2024)
Xin Yu, Ze Yuan, Yuan-Chen Guo, Ying-Tian Liu, Jianhui Liu, Yangguang Li, Yan-Pei Cao, Ding Liang, and Xiaojuan Qi. 2024.
TEXGen: a Generative Diffusion Model for Mesh Textures.
*ACM Trans. Graph.* 43, 6, Article 213 (2024), 14 pages.
[doi:10.1145/3687909](https://doi.org/10.1145/3687909)
- Yu et al. (2025a)
Zhengming Yu, Zhiyang Dou, Xiaoxiao Long, Cheng Lin, Zekun Li, Yuan Liu, Norman Müller, Taku Komura, Marc Habermann, Christian Theobalt, et al. 2025a.
Surf-D: Generating High-Quality Surfaces of Arbitrary Topologies Using Diffusion Models. In *European Conference on Computer Vision*. Springer, 419–438.
- Zhang et al. (2017)
Chao Zhang, Sergi Pujades, Michael J Black, and Gerard Pons-Moll. 2017.
Detailed, accurate, human shape estimation from clothed 3D scan sequences. In *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*. 4191–4200.
- Zhang et al. (2025)
Diyang Zhang, Zhendong Wang, Zegao Liu, Xinming Pei, Weiwei Xu, and Huamin Wang. 2025.
Physics-inspired Estimation of Optimal Cloth Mesh Resolution. In *Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers* *(SIGGRAPH Conference Papers ’25)*. Association for Computing Machinery, New York, NY, USA, Article 23, 11 pages.
- Zhang et al. (2024)
Longwen Zhang, Ziyu Wang, Qixuan Zhang, Qiwei Qiu, Anqi Pang, Haoran Jiang, Wei Yang, Lan Xu, and Jingyi Yu. 2024.
CLAY: A Controllable Large-scale Generative Model for Creating High-quality 3D Assets.
*ACM Trans. Graph.* 43, 4, Article 120 (July 2024), 20 pages.
[doi:10.1145/3658146](https://doi.org/10.1145/3658146)
- Zhao et al. (2025)
Zibo Zhao, Zeqiang Lai, Qingxiang Lin, Yunfei Zhao, Haolin Liu, Shuhui Yang, Yifei Feng, Mingxin Yang, Sheng Zhang, Xianghui Yang, et al. 2025.
Hunyuan3d 2.0: Scaling diffusion models for high resolution textured 3d assets generation.
*arXiv preprint arXiv:2501.12202* (2025).
- Zhou et al. (2024)
Feng Zhou, Ruiyang Liu, Chen Liu, Gaofeng He, Yong-Lu Li, Xiaogang Jin, and Huamin Wang. 2024.
Design2GarmentCode: Turning Design Concepts to Tangible Garments Through Program Synthesis.
*arXiv preprint arXiv:2412.08603* (2024).
- Zhu et al. (2020)
Heming Zhu, Yu Cao, Hang Jin, Weikai Chen, Dong Du, Zhangye Wang, Shuguang Cui, and Xiaoguang Han. 2020.
Deep fashion3d: A dataset and benchmark for 3d garment reconstruction from single images. In *Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28, 2020, Proceedings, Part I 16*. Springer, 512–530.
- Zhu et al. (2022)
Heming Zhu, Lingteng Qiu, Yuda Qiu, and Xiaoguang Han. 2022.
Registering explicit to implicit: Towards high-fidelity garment mesh reconstruction from single images. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 3845–3854.
- Zou et al. (2023)
Xingxing Zou, Xintong Han, and Waikeung Wong. 2023.
CLOTH4D: A Dataset for Clothed Human Reconstruction. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 12847–12857.