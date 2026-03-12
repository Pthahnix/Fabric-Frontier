<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - 2.1. Multi-view Diffusion
  - 2.2. Garment Reconstruction
  - 2.3. Differentiable Simulation
- 3. Differentiable Garment Simulation
  - 3.1. Forward Simulation
  - 3.2. Differentiable CIPC
- 4. Method Overview
- 5. Pre-Optimization Steps
  - 5.1. Simulatable Sewing Pattern Generation
    - 5.1.1. Patch Symmetrization
    - 5.1.2. Sewing Pattern Discretization
  - 5.2. Multi-view Image Generation
  - 5.3. Human Body Reconstruction
  - 5.4. Garment Initialization
- 6. Garment Optimization
  - 6.1. Optimization Overview
  - 6.2. Rendering Losses
    - 6.2.1. Garment Mask Loss
      - Initialization of Component Colors
    - 6.2.2. RGB and Normal Rendering Loss
  - 6.3. Geometric Regularizers
    - 6.3.1. Area Ratio Loss
    - 6.3.2. Corner Regularizers
      - Boundary Corner Regularizer
      - Small-Angle Corner Regularizers
    - 6.3.3. Comfort Loss
    - 6.3.4. Laplacian Loss
    - 6.3.5. Seam Losses
  - 6.4. Post-Iteration Processing
  - 6.5. Remeshing
- 7. Post-Optimization Steps
  - 7.1. Texture Generation
    - Tileable Texture Generation via FabricDiffusion
    - In-the-Wild Texture Generation via GPT-4o and FLUX
  - 7.2. Showcase under Human Motions
- 8. Implementation
  - Differentiable Simulation Layer
  - Balancing between Losses
  - Training Time
- 9. Experiments
  - 9.1. Geometry Reconstruction Comparison
    - Benchmark
    - Baselines
    - Results
  - 9.2. Sewing Pattern Evaluation
  - 9.3. Textured Garment Reconstruction and Simulation
    - Test Images
    - Textured Garment Reconstruction
    - Garment Simulation
  - 9.4. Ablation Study
    - Patch Symmetrization
    - Laplacian Loss
    - Boundary Corner Regularizer
    - Comfort Loss
    - Area Ratio Loss
    - Seam Losses
    - Vertex Color Reconstruction
    - Loose Garments
- 10. Conclusion
  - Limitations and Future Work
  - Ethical Concerns
- References

## Abstract

Abstract. Recent advances in large models have significantly advanced image-to-3D reconstruction. However, the generated models are often fused into a single piece, limiting their applicability in downstream tasks. This paper focuses on 3D garment generation, a key area for applications like virtual try-on with dynamic garment animations, which require garments to be separable and simulation-ready. We introduce Dress-1-to-3, a novel pipeline that reconstructs physics-plausible, simulation-ready separated garments with sewing patterns and humans from an in-the-wild image. Starting with the image, our approach combines a pre-trained image-to-sewing pattern generation model for creating coarse sewing patterns with a pre-trained multi-view diffusion model to produce multi-view images. The sewing pattern is further refined using a differentiable garment simulator based on the generated multi-view images. Versatile experiments demonstrate that our optimization approach substantially enhances the geometric alignment of the reconstructed 3D garments and humans with the input image. Furthermore, by integrating a texture generation module and a human motion generation module, we produce customized physics-plausible and realistic dynamic garment demonstrations. Our project page is https://dress-1-to-3.github.io/ .

## 1. Introduction

Creating digital assets of clothed humans is crucial for a wide range of applications, including virtual reality (VR), the film industry, fashion design, and gaming. However, the traditional pipeline for digital human and garment creation involves multiple intricate steps, such as concept design, material selection, garment modeling, human pose generation, garment fitting, and animation. These processes are often labor-intensive and time-consuming.

In recent years, significant advancements in image-to-3D asset reconstruction have been driven by the development of powerful image and video generation models. Among these, multiview diffusion models [Chen et al., 2024c; Liu et al., 2023a; Gao et al., 2024] have emerged as a promising approach, effectively leveraging multiview images as intermediate representations to capture 3D information. When fine-tuned on human datasets, these models generalize well to avatar reconstructions from in-the-wild images [Li et al., 2024d; He et al., 2024a]. However, the generated results are often fused into a single piece, making them unsuitable for downstream tasks such as garment animation and interaction.

In the meantime, sewing patterns, a foundational representation in the garment design industry, have been adopted as intermediate reconstruction outputs to recover garment geometries [Liu et al., 2023b; Li et al., 2024b]. This representation is particularly advantageous due to its seamless integration with downstream applications such as physics simulation and garment editing. Despite their promise, these feed-forward approaches face significant limitations stemming from the scarcity of high-quality 3D data. As a result, the reconstructed garments are often constrained by the distribution of the training dataset, leading to inaccuracies in aligning with input images. This limitation hinders their ability to produce detailed and diverse reconstructions reflective of real-world garment variations. The question then arises: can we keep the advantages of the simulation-ready representation of sewing patterns while leveraging the powerful priors in large multi-view diffusion models to reconstruct garments from solely an in-the-wild image?

To address this problem, we introduce Dress-1-to-3, a novel garment reconstruction pipeline that accurately transforms an in-the-wild image into a simulation-ready representation of separated human and garment by leveraging the strengths of both 2D multi-view diffusion and 3D sewing pattern reconstruction. To bridge those two parts, we propose a generalized and unified IPC differentiable framework for garment optimization, which enables the optimization of 3D sewing patterns using 2D generative multi-view RGB images and normal maps as guidance. By refining imperfect generative outputs to align with the geometry encoded in multiview images, our approach allows the reconstruction of out-of-distribution garment shapes with high fidelity. Our contributions include:

- •
We propose a holistic garment reconstruction pipeline that takes a single image as input and generates garments fitted onto a posed human, ensuring both the human pose and garments align with the input image.
- •
We derive a generalized and unified IPC differentiable framework that is agonistic to constitutive models. We apply this framework for co-dimensional garment optimization.
- •
We conduct extensive experiments to demonstrate the effectiveness and versatility of our garment optimization framework, successfully reconstructing garments across diverse categories, including those not present in the training dataset.

## 2. Related Work

### 2.1. Multi-view Diffusion

Owing to their powerful predictive ability, Diffusion Probabilistic Models [Ho et al., 2020] have been applied to image [Nichol et al., 2021; Zhang et al., 2023; Dhariwal and Nichol, 2021; Ruiz et al., 2023; Saharia et al., 2022], video [Chen et al., 2024d; Ho et al., 2022], and 3D shape synthesis tasks [Long et al., 2024; Yu et al., 2024b; Tang et al., 2024], etc. However, applying image diffusion models to generate multi-view images separately poses significant challenges in maintaining consistency across different views. To address multi-view inconsistency, multi-view attentions and camera pose controls are adopted to fine-tune pre-trained image diffusion models, enabling the simultaneous synthesis of multi-view images [Shi et al., 2024; Wang and Shi, 2023; Xu et al., 2024; Yang et al., 2024; Shi et al., 2023; Long et al., 2024], though these methods might result in compromised geometric consistency due to the lack of inherent 3D biases. To ensure both global semantic consistency and detailed local alignment in multi-view diffusion models, 3D-adapters [Chen et al., 2024a] propose a plug-in module designed to infuse 3D geometry awareness. Nevertheless, the generated images by these models are sparse views. To address this issue, CAT3D [Gao et al., 2024] introduces an efficient parallel sampling strategy to generate a large set of camera poses, and MVDiffusion++ [Tang et al., 2025] adopts a pose-free architecture and a view dropout strategy to reduce computational costs, generating dense, high-resolution images.

Generating consistent images from multi-view diffusions offers guidance for further 3D shape reconstruction [Gao et al., 2024]. PSHuman [Li et al., 2024d] integrates a body-face cross-scale diffusion with an SMPL-X conditioned multi-view diffusion for clothed human reconstruction with high-quality face details. Recent work, MagicMan [He et al., 2024a], utilizes a hybrid human-specific multi-view diffusion model with 3D SMPL-X-based body priors and 2D diffusion priors to consistently generate dense multi-view RGB images and normal maps, supporting high-quality human mesh reconstruction. Different from these works, we exploited multi-view diffusions to generate multi-view normals and RGB images as guidance to optimize sewing patterns and stitches instead of human meshes.

### 2.2. Garment Reconstruction

Previous work focusing on clothed human reconstruction [Xiu et al., 2022, 2023] typically generates garments fused with digital human models, limiting them to basic skinning-based animations and requiring extra segmentation and editing to separate the garments from the human body. In contrast, our approach focuses on reconstructing separately wearable, simulation-ready garments and human models. Other closely related works include Li et al. [2024a]; Yu et al. [2024a], which also generates simulation-ready clothes via differentiable simulation, but at the cost of creating clothing templates by artists, precise point clouds by scanners or 3D shapes of garments. NeuralTailor [Korosteleva and Lee, 2022] utilizes point-level attention for pattern shape and stitching information regression, enabling the reconstruction of garment meshes from point clouds. In contrast, our paper focuses on reconstructing non-watertight garments and humans separately from a single image without additional inputs.

To reconstruct separated non-watertight garments from a single image, GarVerseLOD [Luo et al., 2024] recovers garment details hierarchically in a coarse-to-fine framework. However, it fails to reconstruct complex skirts or dresses with slits or with complex human poses due to the limited representation of such features in the training data. ClothWild [Moon et al., 2022] exploits a weakly supervised pipeline with DensePose-based loss to further increase robustness on in-the-wild images. BCNet [Jiang et al., 2020] introduces a layered garment representation and a generic skinning weight generation network to model garments with different topologies. Deep Fashion3D [Zhu et al., 2020] refines adaptable templates with rich annotations to fit garment shapes. While they are limited to garment categories in their training datasets, these works fail to reconstruct complex categories such as jumpsuits. Additionally, they require nearly frontal images as input, limiting reconstruction from different views. AnchorUDF [Zhao et al., 2021] explores a learnable unsigned distance function to query both 3D position features and pixel-aligned image features via anchor points, which reconstructs the coarse garment shape but lacks the generation of high-quality geometric details.

Instead of directly reconstructing garment meshes, some works [Liu et al., 2023b; He et al., 2024b; Korosteleva and Sorkine-Hornung, 2023] treat sewing patterns as intermediate representations to generate garments by stitching them together. Recent work, GarmentRecovery [Li et al., 2024b], introduces implicit sewing patterns (ISP) to provide shape priors integrated with deformation priors for further garment recovery, though it builds specialized models for each individual garment or garment type. Both SewFormer [Liu et al., 2023b], and PanelFormer [Chen et al., 2024b] utilize Transformers to predict sewing patterns and stitches. Concurrent work, SewingLDM, AIpparel, Design2GarmentCode, and ChatGarment [Liu et al., 2024; Nakayama et al., 2024; Zhou et al., 2025; Bian et al., 2025], exploits multimodal models to synthesize sewing patterns. However, their garment results lack physical material parameters. Therefore, they fail to reconstruct diverse shapes for garments with different physical materials. While Wang et al. [2018] and Yang et al. [2018] leverage a shared latent space and joint material-pose optimization to generate 3D garments and 2D sewing patterns, their approaches rely heavily on large-scale datasets on garment templates and human-body models, limiting their ability to generalize to out-of-distribution garments and body shapes. Our work aims to generate diverse, image-aligned, simulation-ready garments with high-quality details from in-the-wild images by optimizing sewing patterns and stitches with physical parameters via differentiable simulations.

### 2.3. Differentiable Simulation

Differentiable simulation has seen widespread application in recent research, particularly for system identification and the inference of material parameters from both synthetic [Li et al., 2023a, 2024c] and real-world [Huang et al., 2024; Si et al., 2024] observations. The scope of exploration spans various domains, including fluid dynamics and control [McNamara et al., 2004; Schenck and Fox, 2018; Li et al., 2023b, 2024c], rigid-body dynamics [Freeman et al., 2021; Strecke and Stueckler, 2021; Xu et al., 2023], articulated systems [Geilinger et al., 2020; Qiao et al., 2021; Xu et al., 2021], soft-body dynamics [Hahn et al., 2019; Hu et al., 2019b; Du et al., 2021; Jatavallabhula et al., 2021; Huang et al., 2024], cloth [Li et al., 2022; Stuyck and Chen, 2023; Li et al., 2024a], inelasticity [Huang et al., 2021; Li et al., 2023a], inflatable structures [Panetta et al., 2021], and Voronoi diagrams [Numerow et al., 2024].

Cloth-based applications, whether for static optimization or dynamic simulation [Santesteban et al., 2022; Grigorev et al., 2023], frequently involve extensive frictional contact. Consequently, many works focus on robust methods for handling dry frictional contact in differentiable simulations. Bartle et al. [2016] proposes a physics-driven pattern adjustment for garment editing using fixed-point optimization, which does not account for gradients. Liang et al. [2019] is the first to introduce a fully functional differentiable cloth simulator with frictional contact and self-collision, formulating a quadratic programming problem. Jatavallabhula et al. [2021] employs a penalty-based frictional contact model, while Du et al. [2021] and Li et al. [2022] leverage the adjoint method for Projective Dynamics [Bouaziz et al., 2014] with friction. Building on Position-Based Dynamics [Müller et al., 2007; Macklin et al., 2016], Stuyck and Chen [2023] and Li et al. [2024a] introduce differentiable formulations for compliant constraint dynamics, and Huang et al. [2024] presents an adjoint-based framework for differentiable Incremental Potential Contact (IPC) [Li et al., 2020, 2021].

The finite difference (FD) method [Renardy and Rogers, 2006] is a standard approach to numerical differentiation. The complex-step finite difference technique [Luo et al., 2019; Shen et al., 2021] offers an alternative that mitigates issues such as subtractive cancellation and accumulated numerical errors by leveraging complex Taylor expansions [Brezillon et al., 1981]. They can be used to optimize low-DoF system [Zheng et al., 2025]. Automatic differentiation (AD) [Naumann, 2011; Margossian, 2019] and code transformation libraries like NVIDIA Warp [Macklin, 2022], DiffTaichi [Hu et al., 2019b, a], and others [Herholz et al., 2024] automatically compute gradients based on forward simulation, allowing for greater reuse of existing code. However, they can introduce code constraints, incur a high memory footprint, and may cause gradient explosion if applied naively. Our framework combines NVIDIA Warp’s AD with an adjoint method to achieve both development efficiency and high performance.

## 3. Differentiable Garment Simulation

### 3.1. Forward Simulation

We use Codimensional Incremental Potential Contact (CIPC) [Li et al., 2021] as our underlying garment simulation method, which is the state-of-the-art in cloth simulation regarding accuracy and robustness. It ensures non-penetration through distance-based log barrier energy and continuous collision detection (CCD). Below, we summarize the simulation pipeline, with further details available in Li et al. [2021].

The simulated codimensional surface is discretized into triangles defined by vertices $\bm{V}$ and faces $\bm{F}$. Let $\bm{X}$ denote the vertex positions in the undeformed state, and let $\bm{x}^{n}$ and $\bm{v}^{n}$ represent the vertex positions and velocities, respectively, at time step $t^{n}$. CIPC employs an optimization-based time integrator to achieve the state transition from time step $t^{n}$ to $t^{n+1}=t^{n}+h$, minimizing the following energy:

$$ (1) $\bm{x}^{n+1}=\operatorname*{arg\,min}_{\bm{x}}E(\bm{x})=\frac{1}{2}\|\bm{x}- \tilde{\bm{x}}\|_{\bm{M}}^{2}+\Psi(\bm{x};\bm{X})+B(\bm{x}).$ $$

Here, $\tilde{\bm{x}}=\bm{x}^{n}+\bm{v}^{n}h+\bm{g}h^{2}$ represents the predictive position under backward Euler integration. $\|\cdot\|_{\bm{M}}$ denotes the $L^{2}$-norm weighted by the vertex mass $\bm{M}_{ii}$. $\Psi(\bm{x};\bm{X})$ is the elastic energy, encompassing both stretching and bending energies, depending on the user’s choice. $B(\bm{x})$ is the log barrier energy introduced by IPC, defined over all contacting vertex-triangle and edge-edge pairs. The barrier energy for each pair of primitives increases from zero to infinity as the gap decreases from a threshold $\hat{d}$ to 0 0, providing sufficient repulsion to prevent penetrations.

Newton’s method with line search is employed to solve the optimization problem, requiring the analytical computation of the gradient and Hessian matrix of the energy at each iteration. The step size upper bound in each line search is clamped by CCD to ensure that all intermediate states remain intersection-free, provided that $\bm{x}^{n}$ is initially intersection-free. Finally, the new velocity is updated as $\bm{v}^{n+1}=(\bm{x}^{n+1}-\bm{x}^{n})/h$.

### 3.2. Differentiable CIPC

Huang et al. [2024] provided an analytical derivation of differentiable IPC using the adjoint method. However, their derivation is closely tied to specific choices of constitutive models. To extend their framework to support cloth simulation, tedious derivations of analytical derivatives are required. In this work, we present a simple and unified framework that leverages both automatic differentiation and the adjoint method.

The governing equation of CIPC simulation can be expressed as an implicit nonlinear system of equations derived from the first-order optimality condition of the minimizer for [Equation 1](https://arxiv.org/html/2502.03449v2#S3.E1):

$$ (2) $\displaystyle\bm{G}(\bm{x}^{*};\bm{x}^{n},\bm{v}^{n},\bm{\varsigma}^{n})= \nabla E(\bm{x}^{*};\bm{x}^{n},\bm{v}^{n},\bm{\varsigma}^{n})=\bm{0},$ (3) $\displaystyle\bm{x}^{n+1}=\bm{x}^{*},\quad\bm{v}^{n+1}=\frac{1}{h}(\bm{x}^{*}- \bm{x}^{n}),$ $$

Here, $\bm{x}^{*}$ is the minimizer of the system energy $E$, ${\bm{x}^{n},\bm{v}^{n}}$ represents the last system state, and $\bm{\varsigma}^{n}$ denotes the set of all continuous parameters of the implicit equation, including shape parameters $\bm{X}$, mass matrix $\bm{M}$, elastic moduli, and others. We assume ${\bm{\varsigma}^{n}}$ are independent, although they may share the same values. This abstraction allows the simulator to function as a differentiable layer with ${\bm{x}^{n},\bm{v}^{n},\bm{\varsigma}^{n}}$ as input and ${\bm{x}^{n+1},\bm{v}^{n+1}}$ as output. The computational graph can be handled by any auto-differentiable framework such as PyTorch. The backward operator computes $\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}}$, $\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n}}$, and $\frac{\text{d}\mathcal{L}}{\text{d}\bm{\varsigma}^{n}}$ given $\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n+1}}$ and $\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}}$ for a given training loss function $\mathcal{L}$.

Taking the full derivatives of [Equation 2](https://arxiv.org/html/2502.03449v2#S3.E2) with respect to ${\bm{x}^{n},\bm{v}^{n},\bm{\varsigma}^{n}}$ on both sides, we obtain:

$$ (4) $\frac{\partial\bm{G}}{\partial\bm{x}^{*}}\left[\frac{\text{d}\bm{x}^{*}}{\text {d}\bm{x}^{n}},\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{v}^{n}},\frac{\text{d}\bm {x}^{*}}{\text{d}\bm{\varsigma}^{n}}\right]+\left[\frac{\partial\bm{G}}{ \partial\bm{x}^{n}},\frac{\partial G}{\partial\bm{v}^{n}},\frac{\partial G}{ \partial\bm{\varsigma}^{n}}\right]=\bm{0},$ $$

which leads to

$$ (5) $\left[\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{x}^{n}},\frac{\text{d}\bm{x}^{*}}{ \text{d}\bm{v}^{n}},\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{\varsigma}^{n}} \right]=-\left[\frac{\partial\bm{G}}{\partial\bm{x}^{*}}\right]^{-1}\left[ \frac{\partial\bm{G}}{\partial\bm{x}^{n}},\frac{\partial G}{\partial\bm{v}^{n} },\frac{\partial G}{\partial\bm{\varsigma}^{n}}\right].$ $$

By the chain rule, we have:

$$ (6) $\begin{split}\left[\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}},\frac{\text{ d}\mathcal{L}}{\text{d}\bm{v}^{n}},\frac{\text{d}\mathcal{L}}{\text{d}\bm{ \varsigma}^{n}}\right]=\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n+1}}\left[ \frac{\text{d}\bm{x}^{n+1}}{\text{d}\bm{x}^{n}},\frac{\text{d}\bm{x}^{n+1}}{ \text{d}\bm{v}^{n}},\frac{\text{d}\bm{x}^{n+1}}{\text{d}\bm{\varsigma}^{n}} \right]&\\ +\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}}\left[\frac{\text{d}\bm{v}^{n +1}}{\text{d}\bm{x}^{n}},\frac{\text{d}\bm{v}^{n+1}}{\text{d}\bm{v}^{n}},\frac {\text{d}\bm{v}^{n+1}}{\text{d}\bm{\varsigma}^{n}}\right].&\end{split}$ $$

Here, we assume $\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}}$, $\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n}}$, and $\frac{\text{d}\mathcal{L}}{\text{d}\bm{\varsigma}^{n}}$ are all row vectors to ensure dimension consistency. From [Equation 3](https://arxiv.org/html/2502.03449v2#S3.E3), we have:

$$ (7) $\text{d}\bm{x}^{n+1}=\text{d}\bm{x}^{*},\quad\text{d}\bm{v}^{n+1}=\frac{1}{h}( \text{d}\bm{x}^{*}-\text{d}\bm{x}^{n}).$ $$

Plugging [Equation 7](https://arxiv.org/html/2502.03449v2#S3.E7) into [Equation 6](https://arxiv.org/html/2502.03449v2#S3.E6), we obtain:

$$ (8) $\begin{split}\left[\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}},\frac{\text{ d}\mathcal{L}}{\text{d}\bm{v}^{n}},\frac{\text{d}\mathcal{L}}{\text{d}\bm{ \varsigma}^{n}}\right]=\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n+1}}\left[ \frac{\text{d}\bm{x}^{*}}{\text{d}\bm{x}^{n}},\frac{\text{d}\bm{x}^{*}}{\text{ d}\bm{v}^{n}},\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{\varsigma}^{n}}\right]&\\ +\frac{1}{h}\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}}\left[\frac{\text{ d}\bm{x}^{*}}{\text{d}\bm{x}^{n}}-\bm{I},\frac{\text{d}\bm{x}^{*}}{\text{d}\bm {v}^{n}},\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{\varsigma}^{n}}\right].&\end{split}$ $$

With some rearrangements, we arrive at:

$$ (9) $\begin{split}&\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}}=\left[\frac{\text {d}\mathcal{L}}{\text{d}\bm{x}^{n+1}}+\frac{1}{h}\frac{\text{d}\mathcal{L}}{ \text{d}\bm{v}^{n+1}}\right]\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{x}^{n}}- \frac{1}{h}\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}}\\ &\left[\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n}},\frac{\text{d}\mathcal{L }}{\text{d}\bm{\varsigma}^{n}}\right]=\left[\frac{\text{d}\mathcal{L}}{\text{d }\bm{x}^{n+1}}+\frac{1}{h}\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}} \right]\left[\frac{\text{d}\bm{x}^{*}}{\text{d}\bm{v}^{n}},\frac{\text{d}\bm{x }^{*}}{\text{d}\bm{\varsigma}^{n}}\right].\end{split}$ $$

Denote $\mathcal{A}=\left[\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n+1}}+\frac{1}{h}
\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n+1}}\right]\left[\frac{\partial\bm
{G}}{\partial\bm{x}^{*}}\right]^{-1}$. By [Equation 5](https://arxiv.org/html/2502.03449v2#S3.E5), we have:

$$ (10) $\frac{\text{d}\mathcal{L}}{\text{d}\bm{x}^{n}}=-\mathcal{A}\frac{\partial\bm{G }}{\partial\bm{x}^{n}}-\frac{1}{h}\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n +1}},$ $$

$$ (11) $\left[\frac{\text{d}\mathcal{L}}{\text{d}\bm{v}^{n}},\frac{\text{d}\mathcal{L} }{\text{d}\bm{\varsigma}^{n}}\right]=-\mathcal{A}\left[\frac{\partial\bm{G}}{ \partial\bm{v}^{n}},\frac{\partial\bm{G}}{\partial\bm{\varsigma}^{n}}\right].$ $$

Observe that $\mathcal{A}$ is obtained by solving a linear system, where the coefficient matrix $\frac{\partial\bm{G}}{\partial\bm{x}^{*}}$ is the Hessian matrix of the system energy $E$. The term $\mathcal{A}\left[\frac{\partial\bm{G}}{\partial\bm{x}^{n}},\frac{\partial\bm{G
}}{\partial\bm{v}^{n}},\frac{\partial\bm{G}}{\partial\bm{\varsigma}^{n}}\right]$ back-propagates the differentials in $\mathcal{A}$ to $\bm{x}^{n}$, $\bm{v}^{n}$, and $\bm{\varsigma}^{n}$ through $\bm{G}$, respectively. This process can be implemented by treating $\bm{G}$ as a differentiable layer that supports auto-differentiation. Using AutoDiff, we eliminate the need to manually derive the analytical expressions for $\frac{\partial\bm{G}}{\partial\bm{v}^{n}}$ and $\frac{\partial\bm{G}}{\partial\bm{\varsigma}^{n}}$. All other components required for forward simulations have already been derived.

Figure: Figure 2. Dress-1-to-3 Pipeline. Starting with a single-view input image of a clothed human, we first derive an initial estimation of the sewing pattern. Additionally, we employ multi-view diffusion to generate orbital camera views, which serve as ground-truth 3D information for both human pose and garment shape. Next, we utilize differentiable simulation to sew and drape the pattern onto the posed human model, optimizing its shape and physical parameters in conjunction with geometric regularizers. Finally, the optimized garment shape provides a physically plausible rest shape in its static state and is readily animatable using a physical simulator.
Refer to caption: x2.png

## 4. Method Overview

We start our pipeline by estimating an initial garment sewing pattern from a single-view image. Next, we generate consistent multi-view RGB images and their corresponding normal maps, based on which we predict the human body pose. The 3D garment is initialized by stitching and draping the 2D patterns onto the predicted human model. The garment’s interaction with the human body is simulated using a differentiable CIPC simulator, allowing us to optimize the physical parameters and the shapes of the sewing patterns guided by the previously generated multi-view RGB images, normal maps, and segmentation results. The optimized state produces a simulation-ready scene with a human model wearing well-fitted 3D outfits that align with the input. Garment textures are automatically generated using a visual-language model and image diffusion. Finally, by applying our CIPC simulator, we can simulate dynamic scenes where the predicted human body wears the optimized garments while performing various motion sequences. An illustration of the pipeline is shown in [Figure 2](https://arxiv.org/html/2502.03449v2#S3.F2). We elaborate on each component of the pipeline in the following sections.

## 5. Pre-Optimization Steps

### 5.1. Simulatable Sewing Pattern Generation

From a single-view image, our pipeline starts by generating an initial sewing pattern decomposition along with stitch information using SewFormer [Liu et al., 2023b]. Following SewFormer’s convention, the sewing pattern is represented as a set of quadratic Bézier curves on a 2D plane, forming a collection of disconnected patches. The curves for each patch are connected to form a loop. Let $\mathcal{E}$ denote the set of all curves, with its control parameters comprising the set of curve vertices $\mathcal{P}=\{\bm{P}_{i}\}$ and the set of control points $\mathcal{K}=\{\bm{K}^{e}\}$ for each edge curve $e\in\mathcal{E}$. To enable garment simulation, the patches are discretized into triangle meshes. First, we apply arc-length parameterization to achieve uniform sampling along the patch boundaries. For stitched patch edges, we ensure they share the same number of sampled points. This consistency allows us to apply vertex-to-vertex stitch constraints in garment simulations, simplifying the sewing process. Using the sampled boundary points, we then perform Delaunay triangulation [Shewchuk, 2008] independently for the interior of each patch.

[Uncaptioned image]: x3.png

#### 5.1.1. Patch Symmetrization

The sewing patterns generated by SewFormer often display certain symmetries, which we aim to preserve during garment optimization. SewFormer generates a fixed number of patches with a predefined order for patch names, though some patches may remain inactive. Symmetry information, including self-symmetry and inter-symmetry, is embedded in these patch names. Symmetric edge pairs can be automatically identified by overlapping a patch with its mirrored symmetric counterpart or, in the case of self-symmetry, with the mirrored version of itself. Given the set of symmetric edge pairs $\mathcal{E}_{S}=\{(i,j)\sim(k,l)\}$, we define the validated curve vertices $\{\hat{\bm{P}}_{i}\}$ of the patches prior to triangulation by solving the following quadratic optimization problem:

$$ (12) $\min_{\{\hat{\bm{P}}_{i}\}}\sum_{(i,j)\sim(k,l)}\|(\hat{\bm{P}}_{i}-\hat{\bm{P }}_{j})+\bm{R}_{S}(\hat{\bm{P}}_{k}-\hat{\bm{P}}_{l})\|^{2}_{2}+\epsilon\sum_{ i}\|\hat{\bm{P}_{i}}-\bm{P}_{i}\|^{2}_{2}.$ $$

Here, $\bm{R}_{S}=\begin{bmatrix}-1&0\\
0&1\end{bmatrix}$ represents the flip matrix, assuming the symmetry axis is vertical. This optimization involves solving a fixed-coefficient, positive definite linear system, which ensures differentiability. The validated edge control points $\{\hat{\bm{K}}^{e}\}$ are computed analytically by symmetrizing their relative coordinates. The symmetrization constraints are illustrated in the inset figure. Throughout this paper, we omit the hat notation for validated vertices and control points, as all computations are based on the symmetrized patches. However, it is important to note that the underlying garment optimization variables retain the original, non-symmetry-enforced geometry parameters.

#### 5.1.2. Sewing Pattern Discretization

To enable direct optimization of Bézier curves, we make the sampling from boundary curve parameters to mesh vertices differentiable. Both boundary sampling and interior sampling are conceptualized as fixed-coordinate sampling based on their control points. Each boundary edge curve $e\in\mathcal{E}$ is defined by the starting vertex $\bm{P}_{0}^{e}$, the control point $\bm{K}^{e}$, and the endpoint $\bm{P}_{1}^{e}$ (which is also the starting point of the next edge). The curve can be differentiably parameterized as $\bm{P}^{e}(t)=(1-t)^{2}\bm{P}_{0}^{e}+2(1-t)t\bm{K}^{e}+t^{2}\bm{P}_{1}^{e}$. Uniform sampling along the curve in terms of arc length is represented as a set of parameters $\{t_{1}^{e},\ldots,t_{n_{e}}^{e}\}$, with $\bm{V}^{e}_{i}=\bm{P}^{e}(t_{i}^{e})$ being the sampled points. The number of sampled points $n_{e}$ may vary for different edges. After independent triangulation for each patch, we compute the harmonic coordinate matrix $\bm{H}\in\mathbb{R}^{n_{I}\times n_{B}}$ [Joshi et al., 2007] for all the interior points, where $n_{I}$ is the number of interior vertices and $n_{B}$ is the total number of boundary vertices.
With a slight abuse of notation, we reparameterize the $j$-th interior vertex as
$\bm{V}_{j}^{I}=\sum_{i}\bm{H}_{ji}\bm{V}^{B}_{i}$, with $\bm{H}_{ji}$ denoting its harmonic weight relative to the $i$-th boundary point $\bm{V}_{i}^{B}$. Here $\bm{H}_{ji}$ is zero if $\bm{V}_{j}^{I}$ and $\bm{V}^{B}_{i}$ do not belong to the same patch.
During backpropagation, we fix the boundary sampling coordinates $\bigcup_{e\in\mathcal{E},i\leq n_{e}}\{t_{i}^{e}\}$ and the interior harmonic coordinate matrix $\bm{H}$, so that the triangulation is analytically determined by the original parameters of the Bézier curves. These coordinates are updated only after remeshing is performed, which will be discussed in the garment optimization section.

### 5.2. Multi-view Image Generation

Given a single-view image of a full-body clothed human, we generate a set of multi-view RGB images and normal maps under orbital camera views using a pre-trained multi-view diffusion model, MagicMan [He et al., 2024a]. These multi-view images of the clothed human are treated as ground truth data for human pose and garment shape in the subsequent reconstruction steps.

### 5.3. Human Body Reconstruction

The generated garment is statically draped on a fixed human mesh. To reduce the gap between the reconstructed garment and the image, an accurate human body is required to correctly support the garment. We use SMPL-X [Pavlakos et al., 2019] as our parameterized human model. First, we apply OSX [Lin et al., 2023] to the input single-view image to obtain an initial pose estimation $\bm{\theta}$ and shape estimation $\bm{\beta}$. This initial estimation typically does not perfectly align with other views, and the scaling and rotation are inconsistent across the multi-view images. Subsequently, we fine-tune the pose based on multi-view images using a coarse-to-fine strategy.

In the coarse stage, we estimate joint landmarks on the images using DWPose [Yang et al., 2023]. Here, we optimize only the global scaling $S$ and rotation $\bm{R}$ of the SMPL-X model based on the following landmark loss:

$$ (13) $\mathcal{L}_{\text{Land}}^{\text{P}}=\frac{1}{|\Omega|}\sum_{i}\|\bm{w}_{i} \cdot\left(\operatorname{Proj}(\bm{J}(S,\bm{R},\bm{\theta},\bm{\beta});\Omega_ {i})-\bm{\bar{J}}_{i}\right)\|^{2}_{2},$ $$

where $\Omega=\{\Omega_{i}\}$ represents the set of camera parameters, $\bm{J}$ is the 3D joint location map provided by the SMPL-X model, $\operatorname{Proj}$ is the projection operator from world space to screen space, $\bm{\bar{J}}_{i}$ is the 2D joint location estimated by DWPose, and $\bm{w}_{i}$ is the per-landmark confidence score of the estimation. We use $\|\cdot\|^{2}_{2}$ to denote the mean square error (MSE). This optimization essentially estimates the model-to-world matrix of the SMPL-X model. To further refine pose and shape parameters, in the fine stage, we additionally incorporate the following RGB loss and mask loss:

$$ (14) $\displaystyle\mathcal{L}_{\text{RGB}}^{\text{P}}=\frac{1}{|\Omega|}\sum_{i}\| \bm{(}\bm{M}^{o}_{i})^{c}\cdot(\bm{I}(S,\bm{R},\bm{\theta},\bm{\beta},\bm{C}_{ H};\Omega_{i})-\bar{\bm{I}}_{i})\|_{1},$ (15) $\displaystyle\mathcal{L}_{\text{Mask}}^{\text{P}}=\frac{1}{|\Omega|}\sum_{i}\| (\bm{M}^{o}_{i})^{c}\cdot(\bm{M}(S,\bm{R},\bm{\theta},\bm{\beta};\Omega_{i})- \bar{\bm{M}}_{i})\|_{1}.$ $$

Here, $\bm{C}_{H}$ represents the optimizable human body vertex color, while $\bm{I}(\cdot)$ and $\bm{M}(\cdot)$ denote the posed human body RGB rendering process and contour rendering process under camera view $\Omega_{i}$, implemented using Nvdiffrast [Laine et al., 2020]. ${\bar{\bm{I}}_{i}}$ and ${\bar{\bm{M}}_{i}}$ are the generated multi-view RGB images and masks, respectively. $\bm{M}^{o}_{i}$ represents the occluded region of the human body, which includes the garment region $\bm{M}^{\beta}_{i}$ and other non-garment occlusions $\bm{M}^{\alpha}_{i}$ (such as footwear, accessories, and hair). These regions are generated using SegFormer [Xie et al., 2021]. The notation $(\cdot)^{c}$ denotes the complement of the specified region. We use $\|\cdot\|_{1}$ to denote the mean absolute error (MAE). By excluding the loss computation in the occluded region, we can accommodate loosely fitted garments. In summary, we optimize using the following loss in the fine stage:

$$ (16) $\footnotesize\mathcal{L}^{\text{P}}(S,\bm{R},\bm{\theta},\bm{\beta},\bm{C}_{H} )=\mathcal{L}^{\text{P}}_{\text{RGB}}+\mathcal{L}^{\text{P}}_{\text{Mask}}+ \lambda_{1}\mathcal{L}^{\text{P}}_{\text{Land}}+\lambda_{2}\|\bm{\theta}-\bm{ \theta}_{0}\|_{1}+\lambda_{3}\|\bm{\beta}-\bm{\beta}_{0}\|_{1}.$ $$

Here, we also regularize the pose and shape parameters where $\bm{\theta}_{0}$ and $\bm{\beta}_{0}$ are their initial estimates provided by OSX.

### 5.4. Garment Initialization

The generated sewing patterns are positioned near the human body and sewn together to be dressed. SewFormer provides an initial placement around the T-posed SMPL-X model. To ensure proper layering, we adopt a bottom-to-top strategy for fitting the entire set of garments onto the human body, allowing the top garments to overlay the bottom ones. Connected components are identified by treating stitched vertices as connected. These components are sorted vertically and sequentially fitted from bottom to top through simulations using CIPC. After completing the T-pose fitting, the human body is interpolated from the T-pose to the reconstructed pose, and the entire cloth-human interaction is simulated by treating the human in motion as a moving boundary condition. To secure the bottom garments and prevent them from slipping during pose interpolation, we shrink the rest shape of the triangles near the waist to generate sufficient friction.

## 6. Garment Optimization

### 6.1. Optimization Overview

In garment optimization phase, we iteratively fine tune parameters of sewing pattern so that the statically draped garments on a posed human body match generated multi-view images in all views. We optimize the curve vertex set $\mathcal{P}$ and the control point set $\mathcal{K}$ of Bézier curves using differentiable CIPC simulation based on the generated multi-view images. To further leverage RGB information for assisting the optimization, we also optimize the vertex colors $\bm{C}_{G}$ of the discretized garment mesh for RGB renderings. Additionally, we optimize the global stretching stiffness $\kappa_{s}$ and the global bending stiffness $\kappa_{b}$ to automatically discover a set of physical parameters that align with the 2D observations.

For each optimization iteration, we use CIPC simulation to statically drape the garment onto the fixed-posed human body mesh. Leveraging the robustness of CIPC, we simulate one step of 1 second to directly reach near-static equilibrium. Since the static equilibrium does not locally depend on the initial state, meaning that the Jacobian matrix of the simulated state with respect to the initial state is zero, we update the initial state of iteration $n$, $\bm{x}_{0}^{n}$, to the previously simulated state:

$$ (17) $\bm{x}_{0}^{n}=\operatorname{Sim}(\bm{x}_{0}^{n-1};\bm{\varsigma}(\kappa_{s}, \kappa_{b},\mathcal{P},\mathcal{K})).$ $$

Here, $\operatorname{Sim}$ represents the simulation process described in [section 3](https://arxiv.org/html/2502.03449v2#S3). The initial state, $\bm{x}_{0}^{0}$, is obtained from the initial garment fitting described in [subsection 5.4](https://arxiv.org/html/2502.03449v2#S5.SS4). $\bm{\varsigma}(\mathcal{P},\mathcal{K})$ denotes the simulation rest shape data, including nodal mass, per-stencil elastic stiffness, undistorted material space, and similar properties. To make the simulation as path-independent as possible, we avoid adding friction during the process. To prevent the bottom garments from slipping down, the boundary loop of the bottom component near the waist area is fixed.

In summary, we solve the following optimization problem:

$$ (18) $\min\mathcal{L}(\mathcal{P},\mathcal{K},\kappa_{s},\kappa_{b};\bm{x},\bm{x}_{0 }),$ $$

where $\bm{x}$ represents the simulated state starting from initial state $\bm{x}_{0}$, which is iteratively updated to the previously simulated state. We elaborate on the training losses in $\mathcal{L}$ that we use in the following sections. We observe that edge curvatures $\mathcal{K}$ are more sensitive than vertex positions $\mathcal{P}$. Therefore, we employ a two-stage training approach, where in the first stage, the update of
$\mathcal{K}$ is frozen.

### 6.2. Rendering Losses

#### 6.2.1. Garment Mask Loss

The dominant rendering loss we employ is the garment mask loss. Given the multi-view ground-truth images, we use SegFormer [Xie et al., 2021] to segment top, bottom, and dress garment masks, assigning each component with a distinct color. The mask loss is defined as follows:

$$ (19) $\mathcal{L}_{\text{Mask}}=\frac{1}{|\Omega|}\sum_{i}\|(\bm{M}^{\alpha}_{i})^{c }\cdot(\bm{M}(\bm{x};\bm{C}_{C},\Omega_{i})-\bar{\bm{M}}_{i})\|_{1}.$ $$

Here, $\bm{x}$ represents the simulated state of garments draped over the human body. $\bm{C}_{C}$ denotes the component color, which is discussed in the following section.
The rendered colored mask $\bm{M}(\bm{x};\bm{C}_{C},\Omega_{i})$ is obtained by assigning $\bm{C}_{C}$ to the corresponding garment vertices and setting the human body to black, ensuring that only the non-occluded parts of the garments are rendered. ${\bar{\bm{M}}_{i}}$ is the set of colored garment masks generated from multi-view RGB images. We also exclude the loss computation in the occluded regions ${\bm{M}^{\alpha}_{i}}$ caused by hair and accessories to avoid incorrect mask guidance.

##### Initialization of Component Colors

The component color $\bm{C}_{C}$ is automatically assigned prior to garment optimization. SewFormer typically predicts garments with one or two connected components. We vertically sort the sewn garment components and the 2D mask regions from the first camera view. The component colors are then assigned accordingly. If only one component is predicted but multiple garment masks are present, we adjust the multi-view garment masks to use a single color.

#### 6.2.2. RGB and Normal Rendering Loss

We also utilize RGB and normal rendering losses to improve garment optimization. These losses are introduced to stabilize the training process, as the gradient of the mask rendering loss within the interior regions of the garment is zero. They are formulated similarly to the mask rendering loss:

$$ (20) $\displaystyle\mathcal{L}_{\text{RGB}}=\frac{1}{|\Omega|}\sum_{i}\|\bm{M}^{ \beta}_{i}\cdot(\bm{I}(\bm{x};\bm{C}_{G},\Omega_{i})-\bar{\bm{I}}_{i})\|_{1},$ (21) $\displaystyle\mathcal{L}_{\text{Normal}}=\frac{1}{|\Omega|}\sum_{i}\|\bm{M}^{ \beta}_{i}\cdot(\bm{N}(\bm{x};\Omega_{i})-\bar{\bm{N}}_{i})\|_{1}.$ $$

Here, $\bm{I}(\bm{x};\bm{C}_{G},\Omega_{i})$ represents the garment RGB rendering of the vertex color $\bm{C}_{G}$ under the camera view $\Omega_{i}$, and $\bm{N}(\bm{x};\Omega_{i})$ denotes the corresponding normal map rendering. The sets $\{\bar{\bm{I}}_{i}\}$ and $\{\bar{\bm{N}}_{i}\}$ are the multi-view RGB and normal images generated by the multi-view diffusion process. The loss computation is restricted to the garment regions $\bm{M}^{\beta}_{i}$.

### 6.3. Geometric Regularizers

The sewing pattern optimization under rendering losses alone is ill-posed because, for the same sewn 3D garment mesh, there are infinitely many ways to decompose the mesh into flattened patches. Therefore, we incorporate several geometric losses to regularize the sewing pattern optimization.

#### 6.3.1. Area Ratio Loss

We use the following area ratio loss to preserve the relative area of each patch with respect to the connected component it belongs to:

$$ (22) $\mathcal{L}_{\text{AR}}=\frac{1}{N_{P}}\sum_{p}\left(\frac{\bar{A}_{p}(\bm{X}) }{\bar{A}_{p}(\bm{X}^{0})}-1\right)^{2},$ $$

where $N_{P}$ is the number of garment patches, $\bar{A}_{p}$ is the operator that computes the ratio between the area of the $p$-th patch and the area of the component. $\bm{X}$ represents the current 2D discretization of the garment patches, and $\bm{X}^{0}$ denotes the initial sampling.

#### 6.3.2. Corner Regularizers

[Uncaptioned image]: x4.png

##### Boundary Corner Regularizer

For the boundary loops of garment components, we identify all corner vertices of the original Bézier curves. At these corners, where two patches are typically sewn together, we apply the following boundary corner regularizer to penalize deviations of corner angles from right angles, as illustrated in the inset figure:

$$ (23) $\mathcal{L}_{\text{BC}}=\frac{1}{N_{BC}}\sum_{c}(1-\bm{d}^{c}_{1}\times\bm{d}^ {c}_{2}).$ $$

Here, $N_{BC}$ represents the total number of boundary corners, $\bm{d}^{c}_{1}$ and $\bm{d}^{c}_{2}$ denote two consecutive unit tangent vectors at corner $c$.

##### Small-Angle Corner Regularizers

Small angles at patch corners can introduce instabilities into optimization and simulation; thus, we use the following regularizer to penalize such angles:

$$ (24) $\mathcal{L}_{\text{SAC}}=-\frac{1}{N_{C}}\sum_{c}s_{c}(\bm{X})\widehat{(\bm{V} _{1}^{c}-\bm{V}_{0}^{c})}\times\widehat{(\bm{V}_{2}^{c}-\bm{V}_{0}^{c})}.$ $$

[Uncaptioned image]: x5.png

Here, $N_{C}$ is the number of patch corners, $(\bm{V}_{1}^{c},\bm{V}_{0}^{c},\bm{V}_{2}^{c})$ is the tuple of three consecutive discrete boundary sampling points at the corner $c$, $\widehat{(\cdot)}$ is the vector normalization operator. $s_{c}(\bm{X})$ is a non-differentiable sign function: $s_{c}(\bm{X})=0$ if the discretized corner angle is larger than some threshold, otherwise, $s_{c}(\bm{X})$ equals the sign of the cross product on the initial sewing pattern. This regularization applies to the two cases illustrated in the inset figure. It tries to maintain the same sign of the angle and avoid the angle becoming too small. However, Bézier curves may still intersect at corners even though the discretized corner triangles are normal. We use the following discretization consistency regularizer to align the curve’s end tangents and discrete edge directions:

$$ (25) $\mathcal{L}_{\text{DC}}=\frac{1}{N_{C}}\sum_{c}(2-\bm{\tau}_{1}^{c}\cdot \widehat{(\bm{V}_{1}^{c}-\bm{V}_{0}^{c})}-\bm{\tau}_{2}^{c}\cdot\widehat{(\bm{ V}_{2}^{c}-\bm{V}_{0}^{c})}),$ $$

where $\bm{\tau}_{1}^{c},\bm{\tau}_{2}^{c}$ are two consecutive end tangents of Bézier curves at corner $c$.

#### 6.3.3. Comfort Loss

In addition to the appearance of the fitting matching the observation, we also aim to ensure that the fitting is comfortable. We use the stretching elasticity energy to evaluate the tightness of the fitting. To prevent overly tight fitting, we introduce the following comfort regularizer:

$$ (26) $\mathcal{L}_{\text{Comfort}}=\int\|\bm{F}(\bm{x},\bm{X})-\bm{R}(\bm{F})\|^{2}d \bm{X},$ $$

where $\bm{R}(\bm{F})$ represents the closest rotation matrix to $\bm{F}$. This is the same as the as-rigid-as-possible (ARAP) stretching energy used in the forward simulation, except that here we assume the global stiffness is 1.

#### 6.3.4. Laplacian Loss

To ensure the smoothness of the fitting, we include a Laplacian regularizer:

$$ (27) $\mathcal{L}_{\text{Lap}}=\|\Delta\bm{x}\|_{2},$ $$

where $\Delta$ represents the node-area-weighted Laplacian operator on triangle meshes, and $\bm{x}$ denotes the simulated garment vertex positions.

#### 6.3.5. Seam Losses

The stitched curved edge pairs should have the same shape to prevent undesired wrinkles near the seams. To achieve this, we use a seam length regularization similar to [Li et al., 2024a] to regularize the paired stitched edges:

$$ (28) $\mathcal{L}_{\text{SL}}=\frac{1}{N_{S}}\sum_{e_{i}\sim e_{j}}\left|\int\|\dot{ \bm{P}}^{e_{i}}(t)\|dt-\int\|\dot{\bm{P}}^{e_{j}}(t)\|dt\right|,$ $$

where $N_{S}$ is the number of stitched seams, $e_{i}$ and $e_{j}$ iterate over all stitched edge pairs, and $\dot{\bm{P}}^{e}(t)$ represents the tangent vector. The integral is computed using finite difference and the Riemann sum. Additionally, we regularize the seam curvatures on these pairs to preserve their initial curvatures:

$$ (29) $\mathcal{L}_{\text{SC}}=\frac{1}{2N_{S}}\sum_{e_{i}\sim e_{j}}\|\bar{\mathcal{ K}}^{e_{i}}-\bar{\mathcal{K}}^{e_{i},0}\|+\|\bar{\mathcal{K}}^{e_{j}}-\bar{ \mathcal{K}}^{e_{j},0}\|,$ $$

where $\bar{\mathcal{K}}^{e}$ represents the relative coordinate of the control point within the frame of the curved edge segment $e$, and $\bar{\mathcal{K}}^{e,0}$ denotes its initial value.

### 6.4. Post-Iteration Processing

Occasionally, when two Bézier curves come close to each other—such as when forming a thin strip—the curves may penetrate one another after a parameter update in some iteration. This can lead to flipped triangles, causing the simulation to fail in the next iteration. To address this, we enforce a safeguard that modifies the geometry in-place to prevent such occurrences. Specifically, we optimize the negative triangle areas using a least-squares penalty after each iteration $n$:

$$ (30) $\mathcal{L}_{\text{Flip}}(\mathcal{P},\mathcal{K})=\frac{1}{|F|}\sum_{f}( \epsilon-\min\{A_{f}(\bm{X}),\epsilon\})+\lambda^{\text{Flip}}\|\bm{X}-\bm{X}^ {n+1}\|_{1},$ $$

where $|F|$ is the number of faces $A_{f}$ is the signed area of triangle $f$ and $\bm{X}^{n+1}$ is the discretized garment vertices after the parameter update at iteration $n$. We optimize the above loss only if triangles are close to flipping.

Figure: Figure 3. Sewing Pattern Remeshing. We perform automatic remeshing during optimization when ill-conditioned triangles are detected. To avoid penetration, we pull back the new discretization to the initial unoptimized stage and rerun the garment initialization to fit it onto the human.
Refer to caption: x6.png

### 6.5. Remeshing

During optimization, we use cage deformations defined by a fixed set of harmonic coordinates to deform a fixed number of interior vertices. The triangulation quality can degrade significantly in regions with large deformations, creating challenges for simulations. To address this, we introduce automatic remeshing during the optimization iterations when the mesh quality drops below a predefined threshold. While rerunning the discretization on updated sewing patterns is straightforward, directly remeshing the fitted garment state on the human body can lead to penetrations. This occurs because the underlying smoothly interpolated surface may intersect after re-triangulation, as the collision handling relies on the previous discretization. To resolve this, we propose a refitting procedure that sews and refits the garment patches onto the human body without causing penetrations.

Assume $\bm{\chi}^{0}$ is the initial garment sewing pattern in the continuous domain with triangulation $\mathcal{T}^{0}$. The sewing pattern optimization at step $n$ can be characterized by a map $\Phi^{n}$ from $\bm{\chi}^{0}$ to $\bm{\chi}^{n}$, where $\Phi^{n}$ is a piecewise linear map defined on the continuous domain. Observe that the initial fitting is sewn from the discretization of $\bm{\chi}^{0}$, where SewFormer provides reasonable transformations to position the panels around the human body. After generating a new triangulation $\mathcal{T}^{n}$ of $\bm{\chi}^{n}$, we pull $\mathcal{T}^{n}$ back to $\bm{\chi}^{0}$ as the new triangulation of $\bm{\chi}^{0}$: $\tilde{\mathcal{T}}^{0}\leftarrow[{\Phi^{n}}]^{-1}(\mathcal{T}^{n})$. We then apply the initial transformations to the updated discretization $\tilde{\mathcal{T}}^{0}$ to position the patches around the T-pose human body and execute the fitting procedure described in [subsection 5.4](https://arxiv.org/html/2502.03449v2#S5.SS4). During this fitting process, we set the rest shape as $\tilde{\mathcal{T}}^{0}$. A relaxation process follows, using $\mathcal{T}^{n}$ as the rest shape. The newly fitted results are non-penetrating, and we set them as the initial state $\bm{x}^{n}_{0}$ for the differentiable simulation process. Finally, $\mathcal{T}^{0}$ is replaced with $\tilde{\mathcal{T}}^{0}$. This remeshing process is illustrated in [Figure 3](https://arxiv.org/html/2502.03449v2#S6.F3).

## 7. Post-Optimization Steps

### 7.1. Texture Generation

To complete our pipeline and deliver a fully textured garment directly from a single image input, tailored to the needs of the garment fabrication industry, we incorporate an additional texture generation module. Unlike formulating texture creation as a reconstruction task—an approach constrained by the ill-posed nature of the problem due to sparse inputs, severe distortion, and occlusions caused by the human body and overlapping garment layers—our module adopts generative methods to produce garment textures. This module employs two strategies for texture generation:

##### Tileable Texture Generation via FabricDiffusion

In this strategy, we assume that in real-world garment creation, clothing panels are typically cut from a single piece of fabric and sewn together, resulting in similar and tileable textures. Based on this assumption, given the front-view ground truth input image and its corresponding colored segmentation mask, we identify the largest uniform color square area within the segmentation mask for each garment component (e.g., top or bottom) as the captured texture region. This region may exhibit distortions and varying illumination caused by occlusions and poses in the input image. To address these issues, we process the captured texture region using FabricDiffusion [Zhang et al., 2024], which generates distortion-free and tileable texture maps. To determine the appropriate tiling scale for aligning the textures with the garment’s UV space (optimized in our pipeline), we assume consistent camera view parameters for the front view. This scale can be calculated by multiplying the derivative of the cropped region’s size by a constant factor.

##### In-the-Wild Texture Generation via GPT-4o and FLUX

For generalized textures that do not fall into the above case, we utilize Vision-Language Models (VLMs) in collaboration with a Diffusion model. Specifically, we process the input image using the GPT-4o [Achiam et al., 2023] VLM to extract descriptive keywords for the textures of various components, such as "denim, dark blue, smooth fabric" and "argyle, grey and white, knitted fabric" through prompt-based querying. These extracted keywords are then fed into FLUX [Labs, 2023], which generates the corresponding textures.

### 7.2. Showcase under Human Motions

The reconstructed simulation-ready garment and human model can be used to generate realistic dynamic human motion in clothing using the robust CIPC physics-based simulator. However, IPC-based simulators require the initial configuration of the human model to be penetration-free. As IPC-based simulators produce intersection-free results, self-penetrations of the human model during the given motion can cause solution failures when the human model interacts with garments. To address this issue, we replace the human model during motion with the nearest intersection-free human model by solving Injective Deformation Processing (IDP) [Fang et al., 2021] problems. In solving these IDP problems, we follow the method in Li et al. [2025] where the authors use PBD simulations to resolve collisions between garments and human models. Instead, we apply extended Position-Based Dynamics (XPBD) [Macklin et al., 2016] simulations to resolve self-penetrations in the human model by repelling colliding vertices and faces, while preserving natural deformations.

Figure: Figure 4. Qualitative Comparisons of Geometry Reconstruction. Our proposed method not only generates sewing patterns that seamlessly integrate into animation and simulation workflows but also achieves superior garment reconstruction accuracy compared to baseline methods.
Refer to caption: x7.png

Figure: Figure 5. Qualitative Comparison of Panel Shape Prediction. Neural Tailor [Korosteleva and Lee, 2022] takes ground-truth garment meshes as input, while SewFormer [Liu et al., 2023b] and our proposed method use single-view images as input. Extra unexpected panels and edges with significant errors are highlighted in red.
Refer to caption: x8.png

## 8. Implementation

##### Differentiable Simulation Layer

We implement CIPC simulation using NVIDIA Warp [Macklin, 2022] to utilize the Auto-Diff feature. The simulator is wrapped in a customized autograd.Function to be integrated to the global computational graph.

##### Balancing between Losses

The training loss is weighted sum of all rendering losses and geometric regularizers. We use the $L_{\text{Mask}}$ as the dominant loss and set the relative weights for $L_{\text{RGB}}$ and $L_{\text{Normal}}$ to $\lambda_{\text{RGB}}=0.1$ and $\lambda_{\text{Normal}}=0.1$. For geometric regularizers, we do not have a rule of thumb to balance them. For all experiments, we use $\lambda_{\text{Lap}}=0.001$, $\lambda_{\text{Comfort}}=0.1$, $\lambda_{\text{AR}}=0.01$, $\lambda_{\text{SAC}}=0.01$, $\lambda_{\text{DC}}=0.001$, $\lambda_{\text{BC}}=0.001$, $\lambda_{\text{SL}}=0.1$, $\lambda_{\text{SL}}=0.1$.

##### Training Time

The pre-optimization steps require about 10 minutes. The garment optimization process can be finished within 2 hours on a single RTX 3090 with 24GB device memory.

## 9. Experiments

### 9.1. Geometry Reconstruction Comparison

We first conduct comparison study to evaluate the reconstruction accuracy of baseline methods and our proposed approach.

##### Benchmark

We use the CloSe [Antić et al., 2024] and 4D-Dress [Wang et al., 2024] datasets for the comparison study. CloSe is a large-scale 3D clothing dataset featuring detailed segmentation across diverse clothing classes. 4D-Dress offers high-quality 4D textured scans of dynamic clothed human sequences. For evaluation, we carefully select examples encompassing a variety of human body shapes, poses, and their corresponding front-view images to establish a comprehensive benchmark.

##### Baselines

pare our method with state-of-the-art single-view garment reconstruction methods, including BCNet [Jiang et al., 2020], ClothWild [Moon et al., 2022], GarmentRecovery [Li et al., 2024b], and SewFormer [Liu et al., 2023b]. Among these, BCNet and ClothWild are designed for clothed human reconstruction but are limited to tight-fitting clothing and not readily adaptable for downstream tasks such as animation and simulation. GarmentRecovery extends to loose-fitting garments reconstruction by deforming predicted rest shapes to align with input images. In contrast, SewFormer predicts corresponding sewing patterns directly from images, enabling seamless integration into animation pipelines and physical simulations. Our proposed method builds upon SewFormer and incorporate differentiable simulation to refine 2D panels and physical parameters. For SewFormer and our approach, we simulate the predicted sewing patterns and use the resulting 3D garments for quantitative comparisons.

##### Results

We evaluate the accuracy of baseline methods and our approach using two metrics: Chamfer Distance (CD) and Intersection over Union (IoU). CD quantifies the geometric similarity between reconstructed and ground-truth meshes, while IoU assesses the alignment between the garment mask of the rendered reconstruction and the input front-view images. The quantitative results for the CloSe and 4D-Dress datasets are presented in Table [1](https://arxiv.org/html/2502.03449v2#S9.T1), and visualized qualitative comparisons are shown in Figure [4](https://arxiv.org/html/2502.03449v2#S7.F4).
BCNet and ClothWild tend to produce overly smooth garment meshes, lacking fine wrinkle details. GarmentRecovery improves geometric details but often results in interpenetrated reconstructions. SewFormer predicts sewing patterns that can be directly used for simulation, yet it neglects physical parameters, leading to simulated results that deviate significantly from the ground-truth mesh. In contrast, our method not only generates sewing patterns for seamless integration into downstream pipelines but also optimizes garment physical parameters, enabling accurate geometry reconstruction that closely aligns with ground truth.

**Table 1. Quantitative Comparisons of Geometry Reconstruction. We evaluate the performance of baseline methods and our approach on the CloSe and 4D-Dress datasets. Our proposed method achieves the highest reconstruction accuracy across both datasets.**
| Method | CloSe | 4D-Dress |  |  |
| --- | --- | --- | --- | --- |
| CD$\downarrow$ | IoU$\uparrow$ | CD$\downarrow$ | IoU$\uparrow$ |  |
| BCNet | 2.277 | 0.781 | 4.704 | 0.575 |
| ClothWild | 2.166 | 0.664 | 3.125 | 0.664 |
| GarmentRecovery | 2.058 | 0.831 | 2.983 | 0.776 |
| SewFormer | 2.233 | 0.748 | 2.926 | 0.720 |
| Dress-1-to-3 (Ours) | 1.623 | 0.862 | 2.441 | 0.808 |

### 9.2. Sewing Pattern Evaluation

We compare our method with two approaches that predict garment sewing patterns: Neural Tailor [Korosteleva and Lee, 2022] and SewFormer [Liu et al., 2023b]. Neural Tailor generates sewing patterns from garment point cloud inputs, while SewFormer and our method recover patterns directly from single-view image inputs. For comparison purposes, we sample points from garment meshes to serve as inputs for Neural Tailor. The qualitative results are shown in Figure [5](https://arxiv.org/html/2502.03449v2#S7.F5).
Neural Tailor is trained on garments draped over an average SMPL female body in a T-pose. Consequently, its predictions are usually unsatisfactory if the human pose deviates from the T-pose, and may produce unexpected additional panels. Furthermore, its reliance on garment point clouds as input significantly restricts its practical applicability. SewFormer, on the other hand, generates symmetric and organized panels. However, its predicted panel shapes often fail to align with the input image. For instance, it may predict long pant panels for an image with short pants. This is probably due to the small scale of the dataset used for its training. Nevertheless, collecting a large-scale dataset of real-world clothed human images paired with corresponding garment meshes and sewing patterns is a challenging and resource-intensive task. In contrast, our optimization-based method requires no additional training data. By leveraging differentiable simulation, it refines an initial estimate of the sewing patterns, achieving significantly more accurate results.

### 9.3. Textured Garment Reconstruction and Simulation

Figure: Figure 6. Qualitative Results of Textured Clothed Human. We showcase the generation capability of Dress-1-to-3 using in-the-wild test images from various sources, including both real-world and synthetic images. Our streamlined pipeline generates perfectly fitted 3D garments with visually plausible textures.
Refer to caption: x9.png

##### Test Images

To evaluate the generative capability of our method, we perform extensive tests on a variety of images from different sources including 4D-Dress [Wang et al., 2024], CloSe [Antić et al., 2024] and DeepFashion2 [Ge et al., 2019]. These images exhibit diverse quality levels and human poses, highlighting the robustness of our method. To further extend the generative capability of our method with text prompts, we employ FLUX [Labs, 2023] to generate input images using custom textual descriptions of clothing on a model. For instance, prompts such as "a female model wearing a blazer and pants" are used. To enhance the diversity of the generated results, we randomly incorporate detailed descriptions, including the shape and color of the clothing, as well as the pose and appearance of the model.

##### Textured Garment Reconstruction

As demonstrated in [Figure 6](https://arxiv.org/html/2502.03449v2#S9.F6), Dress-1-to-3 effectively reconstructs 3D garments that accurately fit human models in both real-world and synthetic images. Our method automatically retrieves visually plausible garment textures using image diffusion techniques. This streamlined process requires minimal human effort to reconstruct high-fidelity garments with sewing patterns and offers users the flexibility to easily adjust garment shape and texture.

Figure: Figure 7. Garment Simulation. We animate garment motion using various human sequences as moving boundary conditions. Our simulation-ready garments exhibit physically plausible dynamics.
Refer to caption: x10.png

##### Garment Simulation

The garments synthesized by our method are simulation-ready due to the accurate sewing, fitting, and optimization of garment patterns. The optimized 3D outfits align perfectly with the human body at steady state, avoiding artifacts such as self- or interpenetration. These garments can be seamlessly integrated into physics-based simulations, such as those used in video games. In [Figure 1](https://arxiv.org/html/2502.03449v2#S0.F1) and [Figure 7](https://arxiv.org/html/2502.03449v2#S9.F7), we visualize several simulated human motion sequences showcasing dynamic garment behavior.

### 9.4. Ablation Study

In [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), we perform an ablation study for key individual components in Dress-1-to-3, using the same garment images as in Section [9.3](https://arxiv.org/html/2502.03449v2#S9.SS3). This study evaluates the contributions of each proposed component to the final garment reconstruction quality.

##### Patch Symmetrization

We first evaluate the effectiveness of the proposed patch symmetrization, designed to facilitate better capture of symmetrical geometry. As shown in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), removing symmetry enforcement results in visibly asymmetric outputs compared to the input garment image. This highlights the critical role of symmetry enforcement in preserving structural coherence and alignment, particularly for garments with strong symmetrical patterns, such as dresses or jackets. By aligning the reconstructed mesh to expected symmetrical features, this component ensures geometric fidelity.

##### Laplacian Loss

Laplacian loss $\mathcal{L}_{\text{Lap}}$ is applied to smooth out noise and irregular wrinkles in the reconstructed garment mesh. This loss minimizes high-frequency artifacts, enabling a cleaner and more aesthetically pleasing surface. The weight of $\mathcal{L}_{\text{Lap}}$ is a tunable parameter, allowing users to control the degree of smoothness based on their preferences. As shown in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), a higher weight results in smoother results but may slightly reduce detail, whereas a lower weight preserves intricate wrinkles but may retain noise.

##### Boundary Corner Regularizer

Figure: Figure 8. Ablation Study. We conduct ablation studies on our geometric regularizer to ensure that the sewing pattern maintains both reasonable 2D patterns and a plausible 3D fitted shape. We minimize irregularities such as asymmetry, sharp or acute angles, and inconsistent scaling of the 2D patterns while reducing noisy geometry and unrealistic wrinkles.
Refer to caption: x11.png

The boundary corner regularizer, $\mathcal{L}_{\text{BC}}$, mitigates the occurrence of sharp angles in the reconstructed sewing patterns. Sharp or acute angles can lead to practical difficulties during garment fabrication, as they introduce challenges in stitching and material handling. As demonstrated in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), optimization results obtained without $\mathcal{L}_{\text{BC}}$ often generate sewing patterns with acute or impractical corner geometries, whereas incorporating this regularizer results in smoother, more fabrication-friendly boundaries.

##### Comfort Loss

Comfort loss, $\mathcal{L}_{\text{Comfort}}$, ensures the reconstructed garment mesh adheres to an appropriate scale relative to the input image. This prevents the generation of sewing patterns that are too small or tight, which could compromise wearability. Without $\mathcal{L}_{\text{Comfort}}$, as shown in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), the reconstructed sewing patterns often exhibit significantly smaller dimensions than expected, leading to impractical or unrealistic results. Incorporating this loss ensures that the final garment size aligns with user expectations and real-world usability requirements

##### Area Ratio Loss

To maintain realistic proportions between garment parts, the area ratio loss, $\mathcal{L}_{\text{AR}}$, is applied to ensure that the relative area of each patch remains consistent with the connected components, reflecting real-world fabrication principles. For instance, in a skirt, the front and back panels should have comparable areas to align with practical garment construction. As illustrated in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), omitting $\mathcal{L}_{\text{AR}}$ often results in disproportionate patch sizes, such as an overly large front skirt panel compared to the back, violating fabrication norms.

##### Seam Losses

Two seam losses: the length seam loss, $\mathcal{L}_{\text{SL}}$, and the curvature seam loss, $\mathcal{L}_{\text{SC}}$ are adopted to ensure that stitched curved edge pairs should have the same shape to prevent undesired wrinkles near the seam, and that enforces preservation of seam curvatures, respectively. As shown in [Figure 8](https://arxiv.org/html/2502.03449v2#S9.F8), the absence of $\mathcal{L}_{\text{SL}}$ leads to uneven sleeve seams, introducing visual artifacts and potential fabrication issues. Similarly, without $\mathcal{L}_{\text{SC}}$, the seams can become overly curved, deviating significantly from the intended design. Together, these losses contribute to producing smooth and realistic seams.

Figure: Figure 9. Comparisons between vertex color renderings and texture renderings.
Refer to caption: x12.png

##### Vertex Color Reconstruction

Vertex colors are optimized to assist garment optimization. However, due to the limited mesh resolution, the visual appearance synthesized with vertex colors tends to be overly smooth. Additionally, colors from adjacent parts can bleed into part boundaries. Comparisons between vertex color renderings and texture renderings are shown in [Figure 9](https://arxiv.org/html/2502.03449v2#S9.F9). This necessitates an additional module to generate textures that are not constrained by mesh resolution.

Figure: Figure 10. Our method tries to find a static garment fit that approximates a non-static garment snapshot or a fit under grasping forces. The input images are generated by GPT-4o.
Refer to caption: x13.png

##### Loose Garments

Our method optimizes garments in static fitting states under gravity and body supporting forces. For non-static states, such as a snapshot of flowing, or static states influenced by other external forces such as grasping, our garment optimization process attempts to approximate the non-static states by finding nearby static configurations (as shown in [Figure 10](https://arxiv.org/html/2502.03449v2#S9.F10)). However, these static approximations may not reflect the garment’s true geometry. We acknowledge this as a limitation and leave the extension to dynamic states or broader boundary conditions as future work.

## 10. Conclusion

In this paper, we present a garment reconstruction pipeline, Dress-1-to-3, which takes a single-view image as input and reconstructs a posed human wearing textured garments, with both the human pose and garment shapes closely aligned with the input image. During optimization, we refine the sewing pattern shapes and physical material parameters by leveraging a differentiable CIPC simulator with accurate contact. The resulting garment assets are simulation-ready and can be seamlessly integrated into a physics-based simulator.

We benchmark our pipeline against baseline methods through two key experiments: a quantitative comparison of geometry reconstruction using existing garment datasets and a qualitative evaluation of sewing patterns. In both cases, Dress-1-to-3 significantly outperforms the baseline approaches.

To further assess the Dress-1-to-3’s robustness and performance, we test our textured garment reconstruction using in-the-wild real-world and synthetic images, validated together with animations of dressed humans. The high-quality results demonstrate the robustness and effectiveness of our approach.

Additionally, ablation studies underscore the importance of the patch symmetrization technique and the contributions of each regularization loss term, highlighting their critical role in optimizing the pipeline’s performance.

##### Limitations and Future Work

While our method provides consistent high-fidelity reconstruction and has been extensively tested with in-the-wild images, its generation ability is somewhat limited by the initial estimation of the sewing pattern. For instance, our method cannot predict new connected pattern components if they are not included in the initial estimation. Additionally, challenges arise with layered clothing, as SewFormer can only predict single-layer patterns, causing multi-layered garments to be fused into a single cloth component. It is worth noting that with a more versatile sewing pattern predictor capable of handling such cases, our method would also be able to process more complex garments. We leave this enhancement as future work.

Furthermore, some optimized sewing patterns may not fully adhere to conventional fashion design principles, as our supervision relies solely on ground-truth renderings of fitted garments. Incorporating regularizers based on design conventions in future work could help produce patterns that are more suitable for manufacturing.

Another limitation lies in the overly smoothed garment surfaces produced by our method. To enhance training robustness, we incorporate regularizers such as seam loss and Laplacian loss; however, these also suppress the formation of natural wrinkles. Another contributing factor to the lack of high-frequency detail is the inconsistency in multi-view normal maps generated by MagicMan, which further smooths geometry in detailed regions. Addressing the reconstruction of high-frequency geometric features remains an avenue for future work.

We also observe a gap between input images and generated textures. Since this is not our primary technical contribution, we use off-the-shelf tools for texture generation. Improving PBR texture generation is left for future work, as it warrants a standalone research effort.

Lastly, although our simulation layer supports differentiable dynamic simulation, we currently use it only for static fittings under gravity and body support forces. Extending the garment optimization to handle dynamic scenarios, such as reconstruction from monocular videos, or interaction-driven deformations like grasping, would be both interesting and practically valuable.

##### Ethical Concerns

We acknowledge that body shape biases exist in our input image datasets.

## References

- [1]
- Achiam et al. [2023]
Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya, Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman, Shyamal Anadkat, et al. 2023.
Gpt-4 technical report.
*arXiv preprint arXiv:2303.08774* (2023).
- Antić et al. [2024]
Dimitrije Antić, Garvita Tiwari, Batuhan Ozcomlekci, Riccardo Marin, and Gerard Pons-Moll. 2024.
CloSe: A 3D Clothing Segmentation Dataset and Model. In *2024 International Conference on 3D Vision (3DV)*. IEEE, 591–601.
- Bartle et al. [2016]
Aric Bartle, Alla Sheffer, Vladimir G Kim, Danny M Kaufman, Nicholas Vining, and Floraine Berthouzoz. 2016.
Physics-driven pattern adjustment for direct 3D garment editing.
*ACM Trans. Graph.* 35, 4 (2016), 50–1.
- Bian et al. [2025]
Siyuan Bian, Chenghao Xu, Yuliang Xiu, Artur Grigorev, Zhen Liu, Cewu Lu, Michael J Black, and Yao Feng. 2025.
ChatGarment: Garment Estimation, Generation and Editing via Large Language Models.
*CVPR* (2025).
- Bouaziz et al. [2014]
Sofien Bouaziz, Sebastian Martin, Tiantian Liu, Ladislav Kavan, and Mark Pauly. 2014.
Projective dynamics: Fusing constraint projections for fast simulation.
*ACM transactions on graphics (TOG)* 33, 4 (2014), 1–11.
- Brezillon et al. [1981]
Patrick Brezillon, Jean-François Staub, Anne-Marie Perault-Staub, and Gérard Milhaud. 1981.
Numerical estimation of the first order derivative: approximate evaluation of an optimal step.
*Computers & Mathematics with Applications* 7, 4 (1981), 333–347.
- Chen et al. [2024b]
Cheng-Hsiu Chen, Jheng-Wei Su, Min-Chun Hu, Chih-Yuan Yao, and Hung-Kuo Chu. 2024b.
Panelformer: Sewing Pattern Reconstruction from 2D Garment Images. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*. 454–463.
- Chen et al. [2024a]
Hansheng Chen, Bokui Shen, Yulin Liu, Ruoxi Shi, Linqi Zhou, Connor Z Lin, Jiayuan Gu, Hao Su, Gordon Wetzstein, and Leonidas Guibas. 2024a.
3D-Adapter: Geometry-Consistent Multi-View Diffusion for High-Quality 3D Generation.
*arXiv preprint arXiv:2410.18974* (2024).
- Chen et al. [2024d]
Haoxin Chen, Yong Zhang, Xiaodong Cun, Menghan Xia, Xintao Wang, Chao Weng, and Ying Shan. 2024d.
Videocrafter2: Overcoming data limitations for high-quality video diffusion models. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 7310–7320.
- Chen et al. [2024c]
Zilong Chen, Yikai Wang, Feng Wang, Zhengyi Wang, and Huaping Liu. 2024c.
V3d: Video diffusion models are effective 3d generators.
*arXiv preprint arXiv:2403.06738* (2024).
- Dhariwal and Nichol [2021]
Prafulla Dhariwal and Alexander Nichol. 2021.
Diffusion models beat gans on image synthesis.
*Advances in neural information processing systems* 34 (2021), 8780–8794.
- Du et al. [2021]
Tao Du, Kui Wu, Pingchuan Ma, Sebastien Wah, Andrew Spielberg, Daniela Rus, and Wojciech Matusik. 2021.
Diffpd: Differentiable projective dynamics.
*ACM Transactions on Graphics (TOG)* 41, 2 (2021), 1–21.
- Fang et al. [2021]
Yu Fang, Minchen Li, Chenfanfu Jiang, and Danny M Kaufman. 2021.
Guaranteed globally injective 3D deformation processing.
*ACM Transactions on Graphics* 40, 4 (2021).
- Freeman et al. [2021]
C Daniel Freeman, Erik Frey, Anton Raichuk, Sertan Girgin, Igor Mordatch, and Olivier Bachem. 2021.
Brax–a differentiable physics engine for large scale rigid body simulation.
*arXiv preprint arXiv:2106.13281* (2021).
- Gao et al. [2024]
Ruiqi Gao, Aleksander Holynski, Philipp Henzler, Arthur Brussee, Ricardo Martin-Brualla, Pratul Srinivasan, Jonathan T Barron, and Ben Poole. 2024.
Cat3d: Create anything in 3d with multi-view diffusion models.
*NeurIPS* (2024).
- Ge et al. [2019]
Yuying Ge, Ruimao Zhang, Xiaogang Wang, Xiaoou Tang, and Ping Luo. 2019.
Deepfashion2: A versatile benchmark for detection, pose estimation, segmentation and re-identification of clothing images. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*. 5337–5345.
- Geilinger et al. [2020]
Moritz Geilinger, David Hahn, Jonas Zehnder, Moritz Bächer, Bernhard Thomaszewski, and Stelian Coros. 2020.
Add: Analytically differentiable dynamics for multi-body systems with frictional contact.
*ACM Transactions on Graphics (TOG)* 39, 6 (2020).
- Grigorev et al. [2023]
Artur Grigorev, Michael J Black, and Otmar Hilliges. 2023.
Hood: Hierarchical graphs for generalized modelling of clothing dynamics. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 16965–16974.
- Hahn et al. [2019]
David Hahn, Pol Banzet, James M Bern, and Stelian Coros. 2019.
Real2sim: Visco-elastic parameter estimation from dynamic motion.
*ACM Transactions on Graphics (TOG)* 38, 6 (2019), 1–13.
- He et al. [2024b]
Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, and Lan Xu. 2024b.
Dresscode: Autoregressively sewing and generating garments from text guidance.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–13.
- He et al. [2024a]
Xu He, Xiaoyu Li, Di Kang, Jiangnan Ye, Chaopeng Zhang, Liyang Chen, Xiangjun Gao, Han Zhang, Zhiyong Wu, and Haolin Zhuang. 2024a.
Magicman: Generative novel view synthesis of humans with 3d-aware diffusion and iterative refinement.
*arXiv preprint arXiv:2408.14211* (2024).
- Herholz et al. [2024]
Philipp Herholz, Tuur Stuyck, and Ladislav Kavan. 2024.
A Mesh-based Simulation Framework using Automatic Code Generation.
*ACM Transactions on Graphics (TOG)* 43, 6 (2024), 1–17.
- Ho et al. [2020]
Jonathan Ho, Ajay Jain, and Pieter Abbeel. 2020.
Denoising diffusion probabilistic models.
*Advances in neural information processing systems* 33 (2020), 6840–6851.
- Ho et al. [2022]
Jonathan Ho, Tim Salimans, Alexey Gritsenko, William Chan, Mohammad Norouzi, and David J Fleet. 2022.
Video diffusion models.
*Advances in Neural Information Processing Systems* 35 (2022), 8633–8646.
- Hu et al. [2019a]
Yuanming Hu, Luke Anderson, Tzu-Mao Li, Qi Sun, Nathan Carr, Jonathan Ragan-Kelley, and Frédo Durand. 2019a.
Difftaichi: Differentiable programming for physical simulation.
*arXiv preprint arXiv:1910.00935* (2019).
- Hu et al. [2019b]
Yuanming Hu, Jiancheng Liu, Andrew Spielberg, Joshua B Tenenbaum, William T Freeman, Jiajun Wu, Daniela Rus, and Wojciech Matusik. 2019b.
Chainqueen: A real-time differentiable physical simulator for soft robotics. In *2019 International conference on robotics and automation (ICRA)*. IEEE, 6265–6271.
- Huang et al. [2021]
Zhiao Huang, Yuanming Hu, Tao Du, Siyuan Zhou, Hao Su, Joshua B Tenenbaum, and Chuang Gan. 2021.
Plasticinelab: A soft-body manipulation benchmark with differentiable physics.
*ICLR* (2021).
- Huang et al. [2024]
Zizhou Huang, Davi Colli Tozoni, Arvi Gjoka, Zachary Ferguson, Teseo Schneider, Daniele Panozzo, and Denis Zorin. 2024.
Differentiable solver for time-dependent deformation problems with contact.
*ACM Transactions on Graphics* 43, 3 (2024).
- Jatavallabhula et al. [2021]
Krishna Murthy Jatavallabhula, Miles Macklin, Florian Golemo, Vikram Voleti, Linda Petrini, Martin Weiss, Breandan Considine, Jérôme Parent-Lévesque, Kevin Xie, Kenny Erleben, et al. 2021.
gradsim: Differentiable simulation for system identification and visuomotor control.
*International Conference on Learning Representations (ICLR)* (2021).
- Jiang et al. [2020]
Boyi Jiang, Juyong Zhang, Yang Hong, Jinhao Luo, Ligang Liu, and Hujun Bao. 2020.
Bcnet: Learning body and cloth shape from a single image. In *Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28, 2020, Proceedings, Part XX 16*. Springer, 18–35.
- Joshi et al. [2007]
Pushkar Joshi, Mark Meyer, Tony DeRose, Brian Green, and Tom Sanocki. 2007.
Harmonic coordinates for character articulation.
*ACM transactions on graphics (TOG)* 26, 3 (2007), 71–es.
- Korosteleva and Lee [2022]
Maria Korosteleva and Sung-Hee Lee. 2022.
Neuraltailor: Reconstructing sewing pattern structures from 3d point clouds of garments.
*ACM Transactions on Graphics (TOG)* 41, 4 (2022), 1–16.
- Korosteleva and Sorkine-Hornung [2023]
Maria Korosteleva and Olga Sorkine-Hornung. 2023.
GarmentCode: Programming Parametric Sewing Patterns.
*ACM Transactions on Graphics (TOG)* 42, 6 (2023), 1–15.
- Labs [2023]
Black Forest Labs. 2023.
FLUX.
[https://github.com/black-forest-labs/flux](https://github.com/black-forest-labs/flux).
- Laine et al. [2020]
Samuli Laine, Janne Hellsten, Tero Karras, Yeongho Seol, Jaakko Lehtinen, and Timo Aila. 2020.
Modular Primitives for High-Performance Differentiable Rendering.
*ACM Transactions on Graphics* 39, 6 (2020).
- Li et al. [2025]
Boqian Li, Xuan Li, Ying Jiang, Tianyi Xie, Feng Gao, Huamin Wang, Yin Yang, and Chenfanfu Jiang. 2025.
GarmentDreamer: 3DGS Guided Garment Synthesis with Diverse Geometry and Texture Details.
*3DV* (2025).
- Li et al. [2020]
Minchen Li, Zachary Ferguson, Teseo Schneider, Timothy R Langlois, Denis Zorin, Daniele Panozzo, Chenfanfu Jiang, and Danny M Kaufman. 2020.
Incremental potential contact: intersection-and inversion-free, large-deformation dynamics.
*ACM Trans. Graph.* 39, 4 (2020), 49.
- Li et al. [2021]
Minchen Li, Danny M. Kaufman, and Chenfanfu Jiang. 2021.
Codimensional Incremental Potential Contact.
*ACM Trans. Graph. (SIGGRAPH)* 40, 4, Article 170 (2021).
- Li et al. [2024d]
Peng Li, Wangguandong Zheng, Yuan Liu, Tao Yu, Yangguang Li, Xingqun Qi, Mengfei Li, Xiaowei Chi, Siyu Xia, Wei Xue, et al. 2024d.
PSHuman: Photorealistic Single-view Human Reconstruction using Cross-Scale Diffusion.
*arXiv preprint arXiv:2409.10141* (2024).
- Li et al. [2024b]
Ren Li, Corentin Dumery, Benoît Guillard, and Pascal Fua. 2024b.
Garment Recovery with Shape and Deformation Priors. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 1586–1595.
- Li et al. [2023a]
Xuan Li, Yi-Ling Qiao, Peter Yichen Chen, Krishna Murthy Jatavallabhula, Ming Lin, Chenfanfu Jiang, and Chuang Gan. 2023a.
Pac-nerf: Physics augmented continuum neural radiance fields for geometry-agnostic system identification.
*International Conference on Learning Representations (ICLR)* (2023).
- Li et al. [2024a]
Yifei Li, Hsiao-yu Chen, Egor Larionov, Nikolaos Sarafianos, Wojciech Matusik, and Tuur Stuyck. 2024a.
Diffavatar: Simulation-ready garment optimization with differentiable simulation. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 4368–4378.
- Li et al. [2022]
Yifei Li, Tao Du, Kui Wu, Jie Xu, and Wojciech Matusik. 2022.
Diffcloth: Differentiable cloth simulation with dry frictional contact.
*ACM Transactions on Graphics (TOG)* 42, 1 (2022), 1–20.
- Li et al. [2024c]
Yifei Li, Yuchen Sun, Pingchuan Ma, Eftychios Sifakis, Tao Du, Bo Zhu, and Wojciech Matusik. 2024c.
NeuralFluid: Neural Fluidic System Design and Control with Differentiable Simulation.
*NeurIPS* (2024).
- Li et al. [2023b]
Zhehao Li, Qingyu Xu, Xiaohan Ye, Bo Ren, and Ligang Liu. 2023b.
DiffFR: Differentiable SPH-based fluid-rigid coupling for rigid body control.
*ACM Transactions on Graphics (TOG)* 42, 6 (2023), 1–17.
- Liang et al. [2019]
Junbang Liang, Ming Lin, and Vladlen Koltun. 2019.
Differentiable cloth simulation for inverse problems.
*Advances in Neural Information Processing Systems* 32 (2019).
- Lin et al. [2023]
Jing Lin, Ailing Zeng, Haoqian Wang, Lei Zhang, and Yu Li. 2023.
One-stage 3d whole-body mesh recovery with component aware transformer. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 21159–21168.
- Liu et al. [2023b]
Lijuan Liu, Xiangyu Xu, Zhijie Lin, Jiabin Liang, and Shuicheng Yan. 2023b.
Towards garment sewing pattern reconstruction from a single image.
*ACM Transactions on Graphics (TOG)* 42, 6 (2023), 1–15.
- Liu et al. [2023a]
Ruoshi Liu, Rundi Wu, Basile Van Hoorick, Pavel Tokmakov, Sergey Zakharov, and Carl Vondrick. 2023a.
Zero-1-to-3: Zero-shot one image to 3d object. In *Proceedings of the IEEE/CVF international conference on computer vision*. 9298–9309.
- Liu et al. [2024]
Shengqi Liu, Yuhao Cheng, Zhuo Chen, Xingyu Ren, Wenhan Zhu, Lincheng Li, Mengxiao Bi, Xiaokang Yang, and Yichao Yan. 2024.
Multimodal Latent Diffusion Model for Complex Sewing Pattern Generation.
*arXiv preprint arXiv:2412.14453* (2024).
- Long et al. [2024]
Xiaoxiao Long, Yuan-Chen Guo, Cheng Lin, Yuan Liu, Zhiyang Dou, Lingjie Liu, Yuexin Ma, Song-Hai Zhang, Marc Habermann, Christian Theobalt, et al. 2024.
Wonder3d: Single image to 3d using cross-domain diffusion. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 9970–9980.
- Luo et al. [2019]
Ran Luo, Weiwei Xu, Tianjia Shao, Hongyi Xu, and Yin Yang. 2019.
Accelerated complex-step finite difference for expedient deformable simulation.
*ACM Transactions on Graphics (TOG)* 38, 6 (2019), 1–16.
- Luo et al. [2024]
Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, and Xiaoguang Han. 2024.
GarVerseLOD: High-Fidelity 3D Garment Reconstruction from a Single In-the-Wild Image using a Dataset with Levels of Details.
*ACM Transactions on Graphics (TOG)* 43, 6 (2024), 1–12.
- Macklin [2022]
Miles Macklin. 2022.
Warp: A high-performance python framework for gpu simulation and graphics. In *NVIDIA GPU Technology Conference (GTC)*.
- Macklin et al. [2016]
Miles Macklin, Matthias Müller, and Nuttapong Chentanez. 2016.
XPBD: position-based simulation of compliant constrained dynamics. In *Proceedings of the 9th International Conference on Motion in Games*. 49–54.
- Margossian [2019]
Charles C Margossian. 2019.
A review of automatic differentiation and its efficient implementation.
*Wiley interdisciplinary reviews: data mining and knowledge discovery* 9, 4 (2019), e1305.
- McNamara et al. [2004]
Antoine McNamara, Adrien Treuille, Zoran Popović, and Jos Stam. 2004.
Fluid control using the adjoint method.
*ACM Transactions On Graphics (TOG)* 23, 3 (2004).
- Moon et al. [2022]
Gyeongsik Moon, Hyeongjin Nam, Takaaki Shiratori, and Kyoung Mu Lee. 2022.
3d clothed human reconstruction in the wild. In *European conference on computer vision*. Springer, 184–200.
- Müller et al. [2007]
Matthias Müller, Bruno Heidelberger, Marcus Hennix, and John Ratcliff. 2007.
Position based dynamics.
*Journal of Visual Communication and Image Representation* 18, 2 (2007), 109–118.
- Nakayama et al. [2024]
Kiyohiro Nakayama, Jan Ackermann, Timur Levent Kesdogan, Yang Zheng, Maria Korosteleva, Olga Sorkine-Hornung, Leonidas J Guibas, Guandao Yang, and Gordon Wetzstein. 2024.
AIpparel: A Large Multimodal Generative Model for Digital Garments.
*arXiv preprint arXiv:2412.03937* (2024).
- Naumann [2011]
Uwe Naumann. 2011.
*The art of differentiating computer programs: an introduction to algorithmic differentiation*.
SIAM.
- Nichol et al. [2021]
Alex Nichol, Prafulla Dhariwal, Aditya Ramesh, Pranav Shyam, Pamela Mishkin, Bob McGrew, Ilya Sutskever, and Mark Chen. 2021.
Glide: Towards photorealistic image generation and editing with text-guided diffusion models.
*arXiv preprint arXiv:2112.10741* (2021).
- Numerow et al. [2024]
Logan Numerow, Yue Li, Stelian Coros, and Bernhard Thomaszewski. 2024.
Differentiable voronoi diagrams for simulation of cell-based mechanical systems.
*ACM Transactions on Graphics (TOG)* 43, 4 (2024), 1–11.
- Panetta et al. [2021]
Julian Panetta, Florin Isvoranu, Tian Chen, Emmanuel Siéfert, Benoît Roman, and Mark Pauly. 2021.
Computational inverse design of surface-based inflatables.
*ACM Transactions on Graphics (TOG)* 40, 4 (2021), 1–14.
- Pavlakos et al. [2019]
Georgios Pavlakos, Vasileios Choutas, Nima Ghorbani, Timo Bolkart, Ahmed AA Osman, Dimitrios Tzionas, and Michael J Black. 2019.
Expressive body capture: 3d hands, face, and body from a single image. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*. 10975–10985.
- Qiao et al. [2021]
Yi-Ling Qiao, Junbang Liang, Vladlen Koltun, and Ming C Lin. 2021.
Efficient differentiable simulation of articulated bodies. In *International Conference on Machine Learning*. PMLR, 8661–8671.
- Renardy and Rogers [2006]
Michael Renardy and Robert C Rogers. 2006.
*An introduction to partial differential equations*. Vol. 13.
Springer Science & Business Media.
- Ruiz et al. [2023]
Nataniel Ruiz, Yuanzhen Li, Varun Jampani, Yael Pritch, Michael Rubinstein, and Kfir Aberman. 2023.
Dreambooth: Fine tuning text-to-image diffusion models for subject-driven generation. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 22500–22510.
- Saharia et al. [2022]
Chitwan Saharia, William Chan, Huiwen Chang, Chris Lee, Jonathan Ho, Tim Salimans, David Fleet, and Mohammad Norouzi. 2022.
Palette: Image-to-image diffusion models. In *ACM SIGGRAPH 2022 conference proceedings*. 1–10.
- Santesteban et al. [2022]
Igor Santesteban, Miguel A Otaduy, and Dan Casas. 2022.
Snug: self-supervised neural dynamic garments. IEEE. In *CVF Conference on Computer Vision and Pattern Recognition (CVPR)*, Vol. 2. 9.
- Schenck and Fox [2018]
Connor Schenck and Dieter Fox. 2018.
Spnets: Differentiable fluid dynamics for deep neural networks. In *Conference on Robot Learning*. PMLR, 317–335.
- Shen et al. [2021]
Siyuan Shen, Yin Yang, Tianjia Shao, He Wang, Chenfanfu Jiang, Lei Lan, and Kun Zhou. 2021.
High-Order Differentiable Autoencoder for Nonlinear Model Reduction.
*ACM Trans. Graph.* 40, 4, Article 68 (jul 2021), 15 pages.
- Shewchuk [2008]
Jonathan Richard Shewchuk. 2008.
A two-dimensional quality mesh generator and Delaunay triangulator.
*Computer Science Division University of California at Berkeley, Berkeley, California* (2008), 94720–1776.
- Shi et al. [2023]
Ruoxi Shi, Hansheng Chen, Zhuoyang Zhang, Minghua Liu, Chao Xu, Xinyue Wei, Linghao Chen, Chong Zeng, and Hao Su. 2023.
Zero123++: a single image to consistent multi-view diffusion base model.
*arXiv preprint arXiv:2310.15110* (2023).
- Shi et al. [2024]
Yichun Shi, Peng Wang, Jianglong Ye, Mai Long, Kejie Li, and Xiao Yang. 2024.
Mvdream: Multi-view diffusion for 3d generation.
*ICLR* (2024).
- Si et al. [2024]
Zilin Si, Gu Zhang, Qingwei Ben, Branden Romero, Zhou Xian, Chao Liu, and Chuang Gan. 2024.
DIFFTACTILE: A Physics-based Differentiable Tactile Simulator for Contact-rich Robotic Manipulation.
*ICLR* (2024).
- Strecke and Stueckler [2021]
Michael Strecke and Joerg Stueckler. 2021.
DiffSDFSim: Differentiable rigid-body dynamics with implicit shapes. In *2021 international conference on 3D Vision (3DV)*. IEEE, 96–105.
- Stuyck and Chen [2023]
Tuur Stuyck and Hsiao-yu Chen. 2023.
Diffxpbd: Differentiable position-based simulation of compliant constraint dynamics.
*Proceedings of the ACM on Computer Graphics and Interactive Techniques* 6, 3 (2023), 1–14.
- Tang et al. [2024]
Jiaxiang Tang, Jiawei Ren, Hang Zhou, Ziwei Liu, and Gang Zeng. 2024.
Dreamgaussian: Generative gaussian splatting for efficient 3d content creation.
*ICLR* (2024).
- Tang et al. [2025]
Shitao Tang, Jiacheng Chen, Dilin Wang, Chengzhou Tang, Fuyang Zhang, Yuchen Fan, Vikas Chandra, Yasutaka Furukawa, and Rakesh Ranjan. 2025.
Mvdiffusion++: A dense high-resolution multi-view diffusion model for single or sparse-view 3d object reconstruction. In *European Conference on Computer Vision*. Springer, 175–191.
- Wang and Shi [2023]
Peng Wang and Yichun Shi. 2023.
Imagedream: Image-prompt multi-view diffusion for 3d generation.
*arXiv preprint arXiv:2312.02201* (2023).
- Wang et al. [2018]
Tuanfeng Y Wang, Duygu Ceylan, Jovan Popovic, and Niloy J Mitra. 2018.
Learning a shared shape space for multimodal garment design.
*ACM SIGGRAPH* (2018).
- Wang et al. [2024]
Wenbo Wang, Hsuan-I Ho, Chen Guo, Boxiang Rong, Artur Grigorev, Jie Song, Juan Jose Zarate, and Otmar Hilliges. 2024.
4D-DRESS: A 4D Dataset of Real-World Human Clothing With Semantic Annotations. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 550–560.
- Xie et al. [2021]
Enze Xie, Wenhai Wang, Zhiding Yu, Anima Anandkumar, Jose M Alvarez, and Ping Luo. 2021.
SegFormer: Simple and efficient design for semantic segmentation with transformers.
*Advances in neural information processing systems* 34 (2021).
- Xiu et al. [2023]
Yuliang Xiu, Jinlong Yang, Xu Cao, Dimitrios Tzionas, and Michael J Black. 2023.
Econ: Explicit clothed humans optimized via normal integration. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*. 512–523.
- Xiu et al. [2022]
Yuliang Xiu, Jinlong Yang, Dimitrios Tzionas, and Michael J Black. 2022.
Icon: Implicit clothed humans obtained from normals. In *2022 IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*. IEEE, 13286–13296.
- Xu et al. [2021]
Jie Xu, Tao Chen, Lara Zlokapa, Michael Foshey, Wojciech Matusik, Shinjiro Sueda, and Pulkit Agrawal. 2021.
An end-to-end differentiable framework for contact-aware robot design.
*RSS* (2021).
- Xu et al. [2023]
Jie Xu, Sangwoon Kim, Tao Chen, Alberto Rodriguez Garcia, Pulkit Agrawal, Wojciech Matusik, and Shinjiro Sueda. 2023.
Efficient tactile simulation with differentiability for robotic manipulation. In *Conference on Robot Learning*. PMLR, 1488–1498.
- Xu et al. [2024]
Yinghao Xu, Hao Tan, Fujun Luan, Sai Bi, Peng Wang, Jiahao Li, Zifan Shi, Kalyan Sunkavalli, Gordon Wetzstein, Zexiang Xu, et al. 2024.
Dmv3d: Denoising multi-view diffusion using 3d large reconstruction model.
*ICLR* (2024).
- Yang et al. [2024]
Jiayu Yang, Ziang Cheng, Yunfei Duan, Pan Ji, and Hongdong Li. 2024.
Consistnet: Enforcing 3d consistency for multi-view images diffusion. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 7079–7088.
- Yang et al. [2018]
Shan Yang, Zherong Pan, Tanya Amert, Ke Wang, Licheng Yu, Tamara Berg, and Ming C Lin. 2018.
Physics-inspired garment recovery from a single-view image.
*ACM Transactions on Graphics (TOG)* 37, 5 (2018), 1–14.
- Yang et al. [2023]
Zhendong Yang, Ailing Zeng, Chun Yuan, and Yu Li. 2023.
Effective whole-body pose estimation with two-stages distillation. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 4210–4220.
- Yu et al. [2024a]
Boyang Yu, Frederic Cordier, and Hyewon Seo. 2024a.
Inverse Garment and Pattern Modeling with a Differentiable Simulator. In *Computer Graphics Forum*, Vol. 43. Wiley Online Library, e15249.
- Yu et al. [2024b]
Zhengming Yu, Zhiyang Dou, Xiaoxiao Long, Cheng Lin, Zekun Li, Yuan Liu, Norman Müller, Taku Komura, Marc Habermann, Christian Theobalt, et al. 2024b.
Surf-D: Generating High-Quality Surfaces of Arbitrary Topologies Using Diffusion Models. In *European Conference on Computer Vision*. Springer, 419–438.
- Zhang et al. [2024]
Cheng Zhang, Yuanhao Wang, Francisco Vicente, Chenglei Wu, Jinlong Yang, Thabo Beeler, and Fernando De la Torre. 2024.
FabricDiffusion: High-Fidelity Texture Transfer for 3D Garments Generation from In-The-Wild Images. In *SIGGRAPH Asia 2024 Conference Papers*. 1–12.
- Zhang et al. [2023]
Lvmin Zhang, Anyi Rao, and Maneesh Agrawala. 2023.
Adding conditional control to text-to-image diffusion models. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 3836–3847.
- Zhao et al. [2021]
Fang Zhao, Wenhao Wang, Shengcai Liao, and Ling Shao. 2021.
Learning anchored unsigned distance functions with gradient direction alignment for single-view garment reconstruction. In *Proceedings of the IEEE/CVF International Conference on Computer Vision*. 12674–12683.
- Zheng et al. [2025]
Yang Zheng, Qingqing Zhao, Guandao Yang, Wang Yifan, Donglai Xiang, Florian Dubost, Dmitry Lagun, Thabo Beeler, Federico Tombari, Leonidas Guibas, et al. 2025.
Physavatar: Learning the physics of dressed 3d avatars from visual observations. In *European Conference on Computer Vision*. Springer, 262–284.
- Zhou et al. [2025]
Feng Zhou, Ruiyang Liu, Chen Liu, Gaofeng He, Yong-Lu Li, Xiaogang Jin, and Huamin Wang. 2025.
Design2GarmentCode: Turning Design Concepts to Tangible Garments Through Program Synthesis.
*CVPR* (2025).
- Zhu et al. [2020]
Heming Zhu, Yu Cao, Hang Jin, Weikai Chen, Dong Du, Zhangye Wang, Shuguang Cui, and Xiaoguang Han. 2020.
Deep fashion3d: A dataset and benchmark for 3d garment reconstruction from single images. In *Computer Vision–ECCV 2020: 16th European Conference, Glasgow, UK, August 23–28, 2020, Proceedings, Part I 16*. Springer, 512–530.