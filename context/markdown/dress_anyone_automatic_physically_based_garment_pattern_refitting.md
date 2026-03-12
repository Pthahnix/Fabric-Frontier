<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - Cloth Simulation
  - Differentiable Simulation
  - Pattern Generation.
  - Garment Recovery.
  - Garment Refitting.
- 3. Background
  - 3.1. Garment Simulation
  - 3.2. Differentiable Simulation
- 4. Method
  - 4.1. Target Body Template Fitting
  - 4.2. Target Shape Generation
  - 4.3. Sewing Pattern Optimization
    - Rest Shape Control Cage
  - 4.4. Loss
    - 4.4.1. Target Shape Matching Terms
    - 4.4.2. Boundary Curvature Term
    - 4.4.3. Pattern Matching Term
    - 4.4.4. Total Area Loss
  - 4.5. Pattern Symmetry
- 5. Results
  - 5.1. Comparison to Related Work
  - 5.2. Ablation Studies
  - 5.3. Performance and Implementation Details
- 6. Limitations and Future Work
- 7. Conclusion
- 8. Acknowledgements
- References

## Abstract

Abstract. Well-fitted clothing is essential for both real and virtual garments to enable self-expression and accurate representation for a large variety of body types. Common practice in the industry is to provide a pre-made selection of distinct garment sizes such as small, medium and large. While these may cater to certain groups of individuals that fall within this distribution, they often exclude large sections of the population. In contrast, individually tailored clothing offers a solution to obtain custom-fit garments that are tailored to each individual. However, manual tailoring is time-consuming and requires specialized knowledge, prohibiting the approach from being applied to produce fitted clothing at scale. To address this challenge, we propose a novel method leveraging differentiable simulation for refitting and draping 3D garments and their corresponding 2D pattern panels onto a new body shape, enabling a workflow where garments only need to be designed once, in a single size, and they can be automatically refitted to support numerous body size and shape variations. Our method enables downstream applications, where our optimized 3D drape can be directly ingested into game engines or other applications. Our 2D sewing patterns allow for accurate physics-based simulations and enables manufacturing clothing for the real world.

## 1. Introduction

Clothing is an essential aspect of daily life. Given the vast population, the majority of the garment manufacturing industry primarily focuses on producing clothes in a limited series of discrete sizes (e.g. small, medium and large) rather than custom-made pieces for individuals. Although these garments may not provide a perfect fit, they are more economical and satisfactory for a large portion of the population. However, this approach limits the types of fabrics used and is not inclusive of all possible body variations — certain individuals will require additional tailoring. Similarly, a discrete set of predetermined sizes do not translate well to digital characters where the character body shapes and sizes can vary even more widely. In fact, virtual representations may even be fantastical of nature. Relying on preset clothing sizes will not work on many such characters. While digital artists possess the capability to manually refit these garments, such a task is laborious, time consuming, requiring both access to, and proficiency in specialized software. Therefore, there is a high demand for automated tailoring algorithms that are capable of refitting garments to a wide variety of body types.

To address this, we propose a computational technique, which creates custom-fitted garments for any humanoid character model by adjusting the 2D sewing patterns of the garment directly. By adjusting the 2D sewing patterns in conjunction with the draped 3D model, rather than solely manipulating the 3D clothing mesh, it becomes possible to replicate these garments physically. It also provides ground-truth rest shape information for downstream applications such as cloth simulation, virtual try-on, fashion design and telepresence applications where garments need to be adapted automatically at scale to various avatar representations.
Therefore, making it a valuable tool for fashion designers and animators alike. Our complete representation of combined physically draped 3D garments and associated 2D patterns also enables other post-processing pipelines such as garment remeshing where important information such as seam line locations can be preserved in the remeshed topology. We demonstrate effectiveness across a diverse range of garments and body shapes ranging from humanoid to fantastical which do not conform to humanoid proportions. We demonstrate the efficacy of our method for both loose and tight-fitting clothing. In summary, our main contributions are

- •
An end-to-end garment refitting method that uses differentiable simulation to produce physically-simulated 3D draped garments, complete with their corresponding 2D sewing patterns. Our approach can refit to different body shapes and sizes with varying body mesh topologies.
- •
A well-designed control cage formulation for 2D pattern optimization for which we show that it outperforms recent state-of-the-art methods (Wang, 2018; Li et al., 2024a).
- •
A carefully selected combination of loss function components which enable high quality, design preserving results on a variety of clothing items ranging from tight to loose items.

## 2. Related Work

##### Cloth Simulation

Physics-based simulation of clothing has made incredible strides in the last decades starting with the seminal work of Terzopoulos et al. (1987) and that of Baraff and Witkin (1998) on implicit integration enabling stable simulations with large time steps. Since then, many novel methods (Etzmußet al., 2003; Choi and Ko, 2002) have been proposed such as the optimization formulation of implicit Euler (Martin et al., 2011; Gast et al., 2015). Different approaches provide varying trade-offs between stability, speed and accuracy such as Projective Dynamics (Bouaziz et al., 2014), Position Based Dynamics (Müller et al., 2007) and its extension XPBD (Macklin et al., 2016) and most recently Vertex Block Descent (Chen et al., 2024a). Guo et al. (2018) presented a novel approach leveraging the material point method to simulate thin shells. Stuyck (2022) provides an overview.

##### Differentiable Simulation

There has been a renewed interest in the development of differentiable simulation methods for the purpose of inverse design, system identification (Larionov et al., 2022; Chen et al., 2022a) and for integration with learning-based frameworks (Liang et al., 2019) to allow for seamless gradient propagation between simulation models and neural networks. Early work proposed the use of the Adjoint method to differentiate through implicit simulation of clothing (Wojtan et al., 2006) with applications to keyframe control. This idea has been applied to several other simulation frameworks such as DiffXPBD (Stuyck and Chen, 2023) and DiffPD (Du et al., 2021), which has been extended by DiffCloth (Li et al., 2022) to address frictional contact for clothing.

##### Pattern Generation.

High quality clothing patterns are key for accurate clothing simulations as they contain ground truth rest shape information which is often compromised in the 3D drape due to strain in the material. Additionally, they are required for fabrication and they enable custom tailored clothing design using computer systems. Several works have focused on pattern representation and generation using synthetic data. NeuralTailor (Korosteleva and Lee, 2022) presents a learning-based pipeline and a unified model for different garment types, which allows estimating sewing patterns from 3D point clouds. Follow up work GarmentCode (Korosteleva and Sorkine-Hornung, 2023) presents a programming-based framework for garment pattern construction. DressCode (He et al., 2024) introduces a GPT-based architecture for generating sewing patterns with text guidance. ISP (Li et al., 2024b) focuses on multi-layered clothing where sewing patterns are represented as signed distance fields. Given 3D garments, patterns can be extracted using computational methods (Pietroni et al., 2022; Bang et al., 2021).

##### Garment Recovery.

Recently, several advances have been proposed for recovering clothing items from limited real data such as images (Halimi et al., 2022; Yang et al., 2018) and scans (Li et al., 2024a). Numerous works (Chen et al., 2024b; Liu et al., 2023; Chen et al., 2022b) present deep neural networks to parameterize the space of garment sewing patterns, which allows them to predict sewing patterns from images of clothing. Similarly, Li et al. (2023) introduce a fitting method that leverages shape and deformation priors derived from synthetic data to obtain 3D garment reconstruction from static images. Sarafianos et al. (2024) presents a method to generate simulation-ready 3D clothing from images or text prompts leveraging generative neural networks. DiffAvatar (Li et al., 2024a) uses differentiable simulation to optimize the 2D garment pattern to recover clothing assets from static scans of clothed people. Follow up work PhysAvatar (Zheng et al., 2024) extends this work to enable avatar recovery from multi-video data. A separate line of work focuses on garment appearance recovery (Xiang et al., 2022).

##### Garment Refitting.

A popular line of research focuses on refitting garments from one body shape to another target shape. Some methods operate on the 3D shape of the garment directly (de Goes et al., 2020) leveraging an iterative optimization approach. Brouet et al. (2012) proposed a geometric constrained optimization problem that produces refitted 2D patterns but does so without considering simulated draping effect. Wang (2018) also uses differentiable simulation to optimize for 2D patterns, but its functionality has only been demonstrated to work on a smaller range of body shape variations compared to our work. Bartle et al. (2016) proposed a 2D pattern optimization procedure which enables direct edits to the 3D garment geometry. Recent work considers body movement and its effect on personalized garment fits (Wolff et al., 2023).

Compared to prior research, our method is the first to demonstrate refitting with corresponding 2D patterns, taking into account physical drape across a broad spectrum of body shapes, rather than a limited set of pre-determined template bodies.

## 3. Background

We use differentiable cloth simulation to drape a 3D garment on a static body shape in A-pose. This allows us to obtain a physically-based drape that is in an equilibrium state resting on the body under external forces. The rest shape of the 3D triangles are provided by the 2D sewing pattern. We provide a concise overview of garment simulation, including its differentiable formulation and the subsequent application in optimizing 2D sewing patterns.

### 3.1. Garment Simulation

Our contributions can be used in combination with any differentiable simulation method. Here, we use XPBD (Macklin et al., 2016) to produce the garment drapes on a given body shape because it provides fast and stable results.

We formulate the energy $U(\mathbf{x})$ as a set of constraints functions $\mathbf{C}=\left[C_{1}(\mathbf{x}),\cdot\cdot\cdot,C_{m}(\mathbf{x})\right]^{\top}$ and inverse compliance matrix $\bm{\alpha}^{-1}$ as

$$ (1) $U(\mathbf{x})=\frac{1}{2}\mathbf{C}(\mathbf{x})^{\top}\bm{\alpha}^{-1}\mathbf{ C}(\mathbf{x}).$ $$

The system is solved iteratively with the introduction of constraint multipliers $\bm{\lambda}$ which is computed under the implicit Euler time scheme as

$$ (2) $\displaystyle(\nabla\mathbf{C}(\mathbf{x})^{\top}\mathbf{M}^{-1}\nabla\mathbf{ C}(\mathbf{x})+\tilde{\bm{\alpha}})\Delta\bm{\lambda}=-\mathbf{C}(\mathbf{ \mathbf{x}})-\tilde{\bm{\alpha}}\bm{\lambda},$ $$

where $\tilde{\bm{\alpha}}=\bm{\alpha}/\Delta t^{2}$, and $\mathbf{M}$ is the mass matrix. Given $\Delta\bm{\lambda}$, the position update is then computed as

$$ (3) $\displaystyle\Delta\mathbf{x}=\mathbf{M}^{-1}\nabla\mathbf{C}(\mathbf{x}) \Delta\bm{\lambda}$ $$

With external forces denoted by $\mathbf{f}_{\text{ext}}$ acting on the system, the state $\mathbf{q}_{n}=\left(\mathbf{x}_{n},\mathbf{v}_{n}\right)$ at time step $n$ consisting of positions $\mathbf{x}\in\mathbb{R}^{3V}$ and velocities $\mathbf{v}\in\mathbb{R}^{3V}$ is updated as

$$ (4) $\displaystyle\mathbf{x}_{n+1}$ $\displaystyle=\mathbf{x}_{n}+\Delta\mathbf{x}\left(\mathbf{x}_{n+1}\right)+ \Delta t\left(\mathbf{v}_{n}+\Delta t\mathbf{M}^{-1}\mathbf{f}_{\text{ext}}\right)$ $\displaystyle\mathbf{v}_{n+1}$ $\displaystyle=\frac{\tau}{\Delta t}\left(\mathbf{x}_{n+1}-\mathbf{x}_{n}\right)$ $$

Note that compared to the original formulation, we incorporate additional velocity damping to ensure that the garment attains a stable drape on the body, where $\tau$ is the velocity damping coefficient, which is set to 0.95 in our simulations.

### 3.2. Differentiable Simulation

Our work largely follows the structure proposed in DiffXPBD (Stuyck and Chen, 2023) which provides a differentiable formulation of the XPBD simulation method. To optimize the loss function $\mathcal{L}$ over the control variables $\mathbf{u}$, we compute the gradient through the full dynamic sequence. We leverage the Adjoint method for efficient gradient computation by first computing intermediate Adjoint states $\hat{\mathbf{q}}_{n}=(\hat{\mathbf{x}}_{n}\in\mathbb{R}^{3V},\hat{\mathbf{v}}_
{n}\in\mathbb{R}^{3V})$ for all simulation steps N. Let $\mathbf{Q}=\left[\mathbf{q}_{0},\cdot\cdot\cdot,\mathbf{q}_{m}\right]$ be the concatenation of states $\mathbf{q}_{n}=(\mathbf{x}_{n},\mathbf{v}_{n})$ over time. Then, we can represent the advancement of the state as $\mathbf{Q}=\mathbf{F}(\mathbf{Q},\mathbf{u})$, where $\mathbf{F}$ contains all of the time step formulae from Eq. [4](https://arxiv.org/html/2405.19148v1#S3.E4).

Following the Adjoint method, the gradient is computed as

$$ (5) $\frac{d\mathcal{L}}{d\mathbf{u}}=\hat{\mathbf{Q}}^{\top}\frac{\partial\mathbf{ F}}{\partial\mathbf{u}}+\frac{\partial\mathcal{L}}{\partial\mathbf{u}},$ $$

where $\hat{\mathbf{Q}}$ is the concatenation of all Adjoint states $\hat{\mathbf{q}}_{n}$. Applying Eq. [5](https://arxiv.org/html/2405.19148v1#S3.E5) to our modified XPBD integration scheme in Eq. [4](https://arxiv.org/html/2405.19148v1#S3.E4) results in a slight variation of the Adjoint state computation

$$ (6) $\displaystyle\hat{\mathbf{x}}_{n}$ $\displaystyle=\hat{\mathbf{x}}_{n+1}+\left(\frac{\partial\Delta\mathbf{x}}{ \partial\mathbf{x}}\right)^{\top}\hat{\mathbf{x}}_{n}+\frac{\tau\hat{\mathbf{v }}_{n}}{\Delta t}-\frac{\tau\hat{\mathbf{v}}_{n+1}}{\Delta t}+\frac{\partial \mathcal{L}}{\partial\mathbf{x}}^{\top}$ $\displaystyle\hat{\mathbf{v}}_{n}$ $\displaystyle=\Delta t\hat{\mathbf{x}}_{n+1}+\frac{\partial\mathcal{L}}{ \partial\mathbf{v}}^{\top},$ $$

where we assume the external forces are not dependent on the position and there are no velocity dependent energy terms. Once all Adjoint states have been computed using Eq. [6](https://arxiv.org/html/2405.19148v1#S3.E6), we can compute the gradient with respect to the control variable $\mathbf{u}$ as presented in Eq. [5](https://arxiv.org/html/2405.19148v1#S3.E5). In practice, this entails computing and storing $\frac{\partial\Delta\mathbf{x}}{\partial\mathbf{u}}$ for any control variable for which gradients are required.

## 4. Method

Figure: Figure 2. Given a draped input garment and body, *Dress Anyone* produces refitted 2D patterns and a 3D draped garment to fit a provided target body shape. We first fit a template body model to the target body to make the method robust against topology differences between body meshes. We the produce a target 3D garment shape (de Goes et al., 2020) which we use as a target shape. We then use differentiable simulation to optimize for a refitted 2D pattern that fits the target body. We leverage a robust and efficient control cage formulation which preserves the garment design. Our refitted garments can be used for several downstream applications such as novel motion generation.
Refer to caption: extracted/5627852/figures/DressAnyonePipeline.jpg

Given a reference garment designed and fitted to a specific body shape and its corresponding 2D sewing pattern, our method automatically optimizes for a custom fitted and physically simulated 3D garment, draped on a new body shape along with its corresponding and optimized 2D sewing patterns. We first generate a target 3D shape for the garment item leveraging prior work (de Goes et al., 2020) (Sec. [4.2](https://arxiv.org/html/2405.19148v1#S4.SS2)) to serve as an optimization target later in the method. We propose an additional statistical body model fitting step to alleviate limitations in prior work (see Sec. [4.1](https://arxiv.org/html/2405.19148v1#S4.SS1)). Given this target shape, we employ differentiable simulation to optimize for the 2D sewing pattern such that the 3D garment, which is draped using differentiable physics-based simulation, matches the target shape as closely as possible (see Sec. [4.3](https://arxiv.org/html/2405.19148v1#S4.SS3)). The 2D sewing pattern is parameterized using a number of control points on the sewing patterns that enforce conformal changes with every update. Our choice of pattern representation significantly aids in regularizing the deformation space to preserve the shape and design intent of the sewing patterns of the garment when being custom fit to novel body shapes and sizes. Fig. [2](https://arxiv.org/html/2405.19148v1#S4.F2) visualizes the different components of our computational method.

### 4.1. Target Body Template Fitting

The target shape generation method (de Goes et al., 2020) requires a common mesh connectivity between body geometries. However, meshes can come from a variety of asset generation sources with varying topologies. Therefore, we fit a statistical body model (Loper et al., 2015) to the target body shape, making our proposed method compatible with novel body topologies. Given the recovered model parameters, we pose the fitted body to the canonical A-pose.

### 4.2. Target Shape Generation

Provided with a garment fitted to a certain body, we first estimate a 3D target shape of the refitted garment to a novel body without accompanying 2D sewing patterns using the refitting pipeline from de Goes et al. (2020). Note that this method only provides a 3D shape without physics-based simulation, which means the garment is not
in any draped equilibrium state;
even more importantly, it does not produce the required 2D patterns. This 3D target shape is subsequently used in our differentiable pipeline as the target drape for which we optimize the garment sewing patterns. Note that because the target shape is not physically simulated, we do not expect to match this shape exactly.

### 4.3. Sewing Pattern Optimization

Our method optimizes for the 2D sewing pattern by minimizing a loss in 3D space through the use of differentiable simulation. For our initial estimate, we establish a global scaling factor by computing the average scale between the area of the target shape and the reference pattern area across each triangle. We then uniformly scale the reference pattern using this factor. At every iteration, we drape the current best estimate of the sewing pattern onto the body and simulate until equilibrium is reached. This allows us to evaluate our loss formulation (Sec. [4.4](https://arxiv.org/html/2405.19148v1#S4.SS4)), which compares the simulated result to our target shape (Sec. [4.2](https://arxiv.org/html/2405.19148v1#S4.SS2)). After loss evaluation, we can compute the gradient of the final simulated state of the 3D vertices with respect to the 2D sewing pattern vertices. This quantity can then be transformed to the control cage vertices, which allows us to update our best estimate through the use of gradient descent.

#### Rest Shape Control Cage

Although it would be possible to optimize for the 2D sewing pattern vertices directly, this would produce noisy results which do not preserve the original design of the garment as documented by Li et al. (2024a). Instead, it is desired to regularize the optimization representation through the use of a lower-dimensional control cage as illustrated in Fig. [2](https://arxiv.org/html/2405.19148v1#S4.F2). Observing that the boundary of the sewing patterns are composed of smooth curves and to constrain the parametric space, we propose to use Green coordinate (Lipman et al., 2008) control points to modify the change in the pattern space. Our selection of the Green coordinate control cage, in contrast to the Harmonic control cage (Wang, 2018) and the mean value coordinate control cage (Li et al., 2024a), imposes restrictions on the degrees of freedom, requiring the patterns to deform in a smooth and conformal way. The pattern is enclosed by the control points such that there are no discontinuity at the boundaries. Given $C$ control points $\bm{\zeta}\in\mathbb{R}^{3C}$ and $V$ vertices, where $C\ll V$, we can express the vertex positions $\bar{\mathbf{x}}$ on the 2D sewing patterns as $\bar{\mathbf{x}}=\bm{W}_{1}\bm{\zeta}+\bm{W}_{2}\mathbf{n}(\bm{\zeta})$,
where $\mathbf{n}$ is the outward normal of the edge of the control cage, and $\mathbf{W}_{1}$ and $\mathbf{W}_{2}$ are $3V\times 3C$ constant weight matrices. We compute the gradient as in Eq. [5](https://arxiv.org/html/2405.19148v1#S3.E5). Therefore, we need to compute $\frac{\partial\mathbf{F}}{\partial\mathbf{\zeta}}$, which requires us to compute $\frac{\partial\Delta\mathbf{x}}{\partial\bm{\zeta}}$. The gradient of each control point is the sum of the weighted gradient from the interior points. We thus find

$$ (7) $\frac{\partial\Delta\mathbf{x}}{\partial\bm{\zeta}}=\frac{\partial\Delta \mathbf{x}}{\partial\bar{\mathbf{x}}}\left(\bm{W}_{1}+\bm{W}_{2}\frac{\partial \mathbf{n}}{\partial\bm{\zeta}}\right)$ $$

where $\frac{\partial\Delta\mathbf{x}}{\partial\bar{\mathbf{x}}}$ is computed by differentiating through the position update of the in-plane stretching and shearing formulation of the cloth triangle constraint.

### 4.4. Loss

We propose a weighted combination of several loss components,
each with its own purpose. We leverage feature matching terms to match the provided 3D target shape as closely as possible. However, doing so without any additional regularizing terms produces low quality garment patterns that no longer preserve the original design of the garment. To produce desired results, we augment our loss function with several regularizing terms.

#### 4.4.1. Target Shape Matching Terms

We observe that the position of the boundary and seam lines play a crucial role in determining the style and fit of an outfit, such as the shoulder line. These elements serve as vital indicators for assessing the quality of a refitted outfit. Consequently, our method strives to minimize the discrepancy in the 3D positions of the boundary and seam lines between the target shape and the simulated result.
In addition, to maintain the fit, our loss functions impose penalties on the difference between the interior points. We aim for a close match at the boundary and the seams. However, given that the target shape is not physically simulated, we allow for some slack in the interior by applying a smaller weight.
The loss is formulated as

$$ (8) $\mathcal{L}_{SM}=\alpha\sum_{x\in\partial\Omega}\lVert\mathbf{x}-\mathbf{x}^{* }\rVert^{2}+\beta\sum_{x\in\text{Seam}}\lVert\mathbf{x}-\mathbf{x}^{*}\rVert^{ 2}+\gamma\sum_{x\in\Omega}\lVert\mathbf{x}-\mathbf{x}^{*}\rVert^{2},$ $$

where $\mathbf{x}^{*}$ is the target position, and $\alpha$, $\beta$, and $\gamma$ are the weights, with $\gamma\ll\alpha,\beta$.

#### 4.4.2. Boundary Curvature Term

Although the Green coordinates guarantee conformal changes of the interior, the curvature at the boundary can be distorted under large displacement of the control points and deviates from the reference design. To alleviate the problem, we impose an additional loss to penalize the change of the curvature on the boundary vertices (Li et al., 2024a; Wang, 2018). We seek a scaled rotation matrix $\mathbf{T}_{i}=s\mathbf{R}_{i}\in\mathbb{R}^{2\times 2}$ at each point $\mathbf{x}_{i}$ with least curvature distortion to its connected boundary edges, $\mathbf{T}_{i}=\arg\min_{\mathbf{T}}||\mathbf{e}_{i1}-\mathbf{T}\bar{\mathbf{e
}}_{i1}||^{2}+||\mathbf{e}_{i2}-\mathbf{T}\bar{\mathbf{e}}_{i2}||^{2}$ with $\mathbf{e}_{i1}=\mathbf{x}_{i+1}-\mathbf{x}_{i}$ and $\mathbf{e}_{i2}=\mathbf{x}_{i-1}-\mathbf{x}_{i}$. The loss is defined as the accumulation of the curvature distortion as

$$ (9) $\mathcal{L}_{\text{curvature}}=\sum_{i\in\partial\Omega}\|(\mathbf{e}_{i1}- \mathbf{T}_{i}\bar{\mathbf{e}}_{i1})+(\mathbf{e}_{i2}-\mathbf{T}_{i}\bar{ \mathbf{e}}_{i2})\|^{2},$ $$

#### 4.4.3. Pattern Matching Term

The optimization on each panel are done independently, and can lead to mismatch pattern boundaries. To ensure the pieces can be sewn together without artifacts such as cloth gathering at the seams, we introduce an additional loss term to enforce the boundary of the corresponding sewing pieces to have the same length.

$$ (10) $\mathcal{L}_{\text{PM}}=\sum_{i\in\text{Seam edges}}\|\mathbf{x}_{i}-\mathbf{x }_{i+1}||^{2}-||\mathbf{x}^{\prime}_{i}-\mathbf{x}^{\prime}_{i+1}\|^{2}$ $$

#### 4.4.4. Total Area Loss

We include an additional area term in our loss, which measures the difference of the target total surface area and the total surface area of the simulated shape of each panel. This term is especially important for loose fitting garments where the gradient of the interior points becomes noisy due to wrinkles and folds.

$$ (11) $\mathcal{L}_{TA}=\sum_{p\in\text{panels}}(\text{A}_{p}-\sum_{i\in\text{T}_{p}} \text{A}_{i})^{2}$ $$

### 4.5. Pattern Symmetry

Most garments are designed to be symmetric, with flip symmetry in the sewing patterns. When desired, we enforce symmetry during our optimization process. We first detect the corresponding pairs of patterns with flip symmetry, and enforce the update to be the average of the gradient on the pair of control points. However, most people have some degree of body asymmetry and therefore, truly custom fits may require intentional asymmetry in garment design.

## 5. Results

Figure: Figure 3. *Animated sequences of refitted clothing.* Our method produces custom fit garments, allowing them to be directly draped and simulated. The incorporation of 2D sewing patterns provides an accurate rest shape which significantly improves garment simulation realism. 3D draped geometries do not provide an accurate rest shape because they frequently contain deformations such as stretching or sagging under gravity which are already integrated into the mesh. This can lead to inaccuracies during the simulation process. Additionally, it allows for our garments to be manufactured from real fabrics. We showcase a select number of frames of a yoga sequence, demonstrating that the fitted garments allow for rich dynamics and diversity in body pose.
Refer to caption: extracted/5627852/figures/newMotion.jpg

We evaluate our method by refitting various clothing items to a wide variety of body shapes. All results are obtained with symmetry enforcement. Our results demonstrate that our method is capable of generalizing to new body shapes, where refitted garments retain their original design intend. We produce simulation-ready garment assets, which enables downstream applications such as the novel animations shown in Fig. [3](https://arxiv.org/html/2405.19148v1#S5.F3).

Figures [1](https://arxiv.org/html/2405.19148v1#S0.F1) and [7](https://arxiv.org/html/2405.19148v1#S8.F7) demonstrate results for a variety of garments, both tight and loose fitting, refitted onto different body shapes and sizes. Note that our method is capable of handling complex garment designs. The garment in Fig. [1](https://arxiv.org/html/2405.19148v1#S0.F1) consists of a total of 28 individual sewing pattern panels. We show examples of refitting garments with significantly non-uniform changes in the body shapes, see for example the pregnant woman in Fig. [7](https://arxiv.org/html/2405.19148v1#S8.F7) or the alien in Fig. [1](https://arxiv.org/html/2405.19148v1#S0.F1) with disproportionate limb sizes where the arms are much shorter compared to the reference. Additionally, notice that in Fig. [8](https://arxiv.org/html/2405.19148v1#S8.F8), the curvy body has a much smaller waist line compared to their upper torso. Yet, our method produces a properly refitted pattern to generate a custom fit. To showcase that our approach produces manufacturable clothing items, we created a T-shirt garment from refitted patterns provided by our method and compare the virtual drape to the real drape in Fig. [6](https://arxiv.org/html/2405.19148v1#S5.F6). The shirt is made from non-stretchable fabric and would highlight any areas of poor fit. However, our result shows that it aligns very well with the target body.

**Table 1. Quantitative Comparisons. Measuring a triangle quality indicator (Shewchuk, 2002), we find that we produce the best results for the refitted sewing patterns shown in Fig. [8](https://arxiv.org/html/2405.19148v1#S8.F8). Higher values indicate better quality.**
|  | Ours | DiffAvatar | (Wang, 2018) |  |
| --- | --- | --- | --- | --- |
| Min $\uparrow$ | 0.263922 | 0.195618 | 0.10746 |  |
| Avg $\uparrow$ | 0.944209 | 0.938846 | 0.944205 |  |

### 5.1. Comparison to Related Work

Our approach is most similar to DiffAvatar but differs in several key aspects. DiffAvatar is designed to fit template garments to incomplete scans of clothed people whereas ours provides an end-to-end method to refit garments onto any body shape producing a 3D drape with associated 2D sewing patterns. Our approach provides better regularization through our choice of control cage formulation and enables symmetry in the produced patterns, resulting in more realistic sewing patterns. As a result, our method’s capability to refit a wider array of body shapes is improved, as demonstrated in Fig. [8](https://arxiv.org/html/2405.19148v1#S8.F8). Using the same loss function, we optimized the sewing patterns with the use of the three different control cages, and only our proposed Green coordinate control cage finds a desirable high-quality result. Since the other two control cages do not guarantee conformal changes, the updated patterns often result in shapes that deviate from the reference design and introduce poorly-shaped narrow triangles, which can lead to numerical instability for both the forward and backward simulation, see Tab. [1](https://arxiv.org/html/2405.19148v1#S5.T1) for a quantitative comparison. As a result these approaches cannot further reduce the loss. We provide a comparison of the 2D pattern quality using the triangle quality metric provided by Shewchuk (2002), and the result shows that our method has the best quality, which is important to physically simulate the refitted garment.

### 5.2. Ablation Studies

We validate our method with ablation studies to verify that the terms in Sec. [4.4](https://arxiv.org/html/2405.19148v1#S4.SS4) assist in producing high quality refits. We demonstrate an ablation of the seam line vertex matching term and the total area matching term on the 3D results in Fig. [4](https://arxiv.org/html/2405.19148v1#S5.F4). This shows that without these terms, the optimization converges to a suboptimal state where it cannot reach the reference shape at the bottom of the pants.
At the top of Fig. [5](https://arxiv.org/html/2405.19148v1#S5.F5)
we compare the 2D patterns with and without enforcing symmetry on the patterns, and it is evident that the patterns are more desirable with the enforced symmetry. Lastly, at the bottom of Fig. [5](https://arxiv.org/html/2405.19148v1#S5.F5), we compare the patterns with and without the loss term on the boundary curvature. Note that while Green coordinates guarantee conformal mapping and thus maintain good triangle quality at largely deformed areas, without the boundary curvature loss the pattern contains less pleasant geometry with a largely curved boundary that is uncommon for regular patterns.

Figure: Figure 4. *3D Ablation.* We provide a visual ablation of the resulting 3D physically simulated drapes when omitting loss term regularizers. Far left shows a visualization of the individual garment panels, highlighting the seam locations used for the seam location matching. We then show the target 3D shape obtained by de Goes et al. (2020) which does not produce 2D sewing patterns. Our full method closely matches the target drape, including the faithful recreation of the ankle cuffs, with custom fit sewing patterns. Note that we do not expect an exact match since the target drape is not physically simulated under external forces such as gravity or body and cloth self collisions. Omitting either of the regularizers produces lower quality fits that do not match the target as closely. Note especially that the ankle cuffs are only preserved with our full approach. The target location of the crotch seam line (highlighted by the red line) is only matched with our full method.
Refer to caption: extracted/5627852/figures/ablationLoss3DV3.jpg

Figure: Figure 5. *2D Ablation.* We assess the importance of our proposed symmetry enforcing approach (ski suit, top) and boundary curvature regularization (pants, bottom) by analyzing the sewing patterns in 2D space. Top : Symmetry enforcement naturally leads to realistic garments patterns to how an experienced tailor would produce them. Bottom : We additionally see the importance of boundary curvature regularization. Omitting this term produces curved pattern boundaries, drastically changing the intended design of the original fit.
Refer to caption: extracted/5627852/figures/ablationLoss2D.jpg

Figure: Figure 6. We demonstrate a real world example of a garment refitted using our approach where we manufactured a physical shirt given the refitted sewing patterns. Left shows the original garment fitted to the original body. Middle shows our virtual refitted garment which is physically draped on the new body. The body geometry was obtained using a 3D scan. Right shows the real garment draped on the real person from whom the scan was taken. Note how the real drape matches the virtual resized garment.
Refer to caption: extracted/5627852/figures/egor3.jpg

### 5.3. Performance and Implementation Details

We implemented our algorithm in C++. All experiments were run on CPU using the AMD Ryzen Threadripper PRO processor. The method takes between 1 to 5 minutes for each optimization iteration, which includes forward simulation and gradient back propagation. The total refitting time depends on optimization iteration count and is typically between 5 to 30 minutes depending on the mesh resolution of the garment.

## 6. Limitations and Future Work

The scope of this work is limited to single layer garments. However, we believe our method to be directly applicable to multi-layer garments provided that the differentiable simulator is able to accurately resolve all cloth interactions. Due to the refitting approach of our method, we preserve the garment design, a limitation of this method is that we can not adapt to virtual characters where additional pattern pieces are required such as adding additional sleeves.

As future work, we would like to include differentiable soft-tissue simulation of the underlying flesh to additionally optimize for comfort when wearing the garments. A natural extension would be to include several different body poses in the optimization loss in order to obtain a better fit under movements as proposed by Wolff et al. (2023). Our proposed end-to-end differentiable physics-based method allows for the direct integration of such physics-based losses into the computational framework.

## 7. Conclusion

We introduced Dress Anyone, a novel computational approach that leverages differentiable simulation and a well-suited pattern representation for optimization to improve on the long standing problem of automated tailoring. Our method incorporates 3D physics-based draping and rest shape optimization to obtain a full representation of garment assets enabling automatic processing for downstream applications. Garments need to be designed only once and can be reused for different body shapes and sizes, including fantastical body types. Our method allows artists to focus their creativity on garment design without burdening them with the laborious and repetitive task of garment refitting.

## 8. Acknowledgements

We would like to thank Romain Prévost for providing us with an implementation of de Goes et al. (2020) and Natasha Devaud for their help with garment design and manufacturing.

## References

- (1)
- Bang et al. (2021)
Seungbae Bang, Maria Korosteleva, and Sung-Hee Lee. 2021.
Estimating garment patterns from static scan data. In *Computer Graphics Forum*, Vol. 40. Wiley Online Library, 273–287.
- Baraff and Witkin (1998)
David Baraff and Andrew Witkin. 1998.
Large steps in cloth simulation. In *Proceedings of the 25th Annual Conference on Computer Graphics and Interactive Techniques* *(SIGGRAPH ’98)*. Association for Computing Machinery, New York, NY, USA, 43–54.
[https://doi.org/10.1145/280814.280821](https://doi.org/10.1145/280814.280821)
- Bartle et al. (2016)
Aric Bartle, Alla Sheffer, Vladimir G Kim, Danny M Kaufman, Nicholas Vining, and Floraine Berthouzoz. 2016.
Physics-driven pattern adjustment for direct 3D garment editing.
*ACM Trans. Graph.* 35, 4 (2016), 50–1.
- Bouaziz et al. (2014)
Sofien Bouaziz, Sebastian Martin, Tiantian Liu, Ladislav Kavan, and Mark Pauly. 2014.
Projective Dynamics: Fusing Constraint Projections for Fast Simulation.
*Acm Transactions On Graphics* 33, 4 (2014), 154.
- Brouet et al. (2012)
Remi Brouet, Alla Sheffer, Laurence Boissieux, and Marie-Paule Cani. 2012.
Design preserving garment transfer.
*ACM Transactions on Graphics* 31, 4 (2012), Article–No.
- Chen et al. (2024a)
Anka He Chen, Ziheng Liu, Yin Yang, and Cem Yuksel. 2024a.
Vertex Block Descent.
*arXiv preprint arXiv:2403.06321* (2024).
- Chen et al. (2024b)
Cheng-Hsiu Chen, Jheng-Wei Su, Min-Chun Hu, Chih-Yuan Yao, and Hung-Kuo Chu. 2024b.
Panelformer: Sewing Pattern Reconstruction From 2D Garment Images. In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*. 454–463.
- Chen et al. (2022a)
Hsiao-yu Chen, Edith Tretschk, Tuur Stuyck, Petr Kadlecek, Ladislav Kavan, Etienne Vouga, and Christoph Lassner. 2022a.
Virtual elastic objects. In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*. 15827–15837.
- Chen et al. (2022b)
Xipeng Chen, Guangrun Wang, Dizhong Zhu, Xiaodan Liang, Philip Torr, and Liang Lin. 2022b.
Structure-Preserving 3D Garment Modeling with Neural Sewing Machines.
*Advances in Neural Information Processing Systems* 35 (2022), 15147–15159.
- Choi and Ko (2002)
Kwang-Jin Choi and Hyeong-Seok Ko. 2002.
Stable but responsive cloth.
*ACM Transactions on Graphics (TOG)* 21, 3 (2002), 604–611.
- de Goes et al. (2020)
Fernando de Goes, Donald Fong, and Meredith O’Malley. 2020.
Garment Refitting for Digital Characters. In *ACM SIGGRAPH 2020 Talks* (Virtual Event, USA) *(SIGGRAPH ’20)*. Association for Computing Machinery, New York, NY, USA, Article 74, 2 pages.
[https://doi.org/10.1145/3388767.3407348](https://doi.org/10.1145/3388767.3407348)
- Du et al. (2021)
Tao Du, Kui Wu, Pingchuan Ma, Sebastien Wah, Andrew Spielberg, Daniela Rus, and Wojciech Matusik. 2021.
Diffpd: Differentiable projective dynamics.
*ACM Transactions on Graphics (TOG)* 41, 2 (2021), 1–21.
- Etzmußet al. (2003)
Olaf Etzmuß, Michael Keckeisen, and Wolfgang Straßer. 2003.
A Fast Finite Element Solution for Cloth Modelling. In *Proceedings of the 11th Pacific Conference on Computer Graphics and Applications* *(PG ’03)*. IEEE Computer Society, USA, 244.
- Gast et al. (2015)
Theodore F Gast, Craig Schroeder, Alexey Stomakhin, Chenfanfu Jiang, and Joseph M Teran. 2015.
Optimization integrator for large time steps.
*IEEE transactions on visualization and computer graphics* 21, 10 (2015), 1103–1115.
- Guo et al. (2018)
Qi Guo, Xuchen Han, Chuyuan Fu, Theodore Gast, Rasmus Tamstorf, and Joseph Teran. 2018.
A material point method for thin shells with frictional contact.
*ACM Transactions on Graphics (TOG)* 37, 4 (2018), 1–15.
- Halimi et al. (2022)
Oshri Halimi, Tuur Stuyck, Donglai Xiang, Timur Bagautdinov, He Wen, Ron Kimmel, Takaaki Shiratori, Chenglei Wu, Yaser Sheikh, and Fabian Prada. 2022.
Pattern-based cloth registration and sparse-view animation.
*ACM Transactions on Graphics (TOG)* 41, 6 (2022), 1–17.
- He et al. (2024)
Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, and Lan Xu. 2024.
DressCode: Autoregressively Sewing and Generating Garments from Text Guidance.
arXiv:2401.16465 [cs.CV]
- Korosteleva and Lee (2022)
Maria Korosteleva and Sung-Hee Lee. 2022.
Neuraltailor: Reconstructing sewing pattern structures from 3d point clouds of garments.
*ACM Transactions on Graphics (TOG)* 41, 4 (2022), 1–16.
- Korosteleva and Sorkine-Hornung (2023)
Maria Korosteleva and Olga Sorkine-Hornung. 2023.
GarmentCode: Programming Parametric Sewing Patterns.
*ACM Transactions on Graphics (TOG)* 42, 6 (2023), 1–15.
- Larionov et al. (2022)
Egor Larionov, Marie-Lena Eckert, Katja Wolff, and Tuur Stuyck. 2022.
Estimating cloth elasticity parameters using position-based simulation of compliant constrained dynamics.
*arXiv preprint arXiv:2212.08790* (2022).
- Li et al. (2023)
Ren Li, Corentin Dumery, Benoît Guillard, and Pascal Fua. 2023.
Garment Recovery with Shape and Deformation Priors.
*arXiv preprint arXiv:2311.10356* (2023).
- Li et al. (2024b)
Ren Li, Benoît Guillard, and Pascal Fua. 2024b.
Isp: Multi-layered garment draping with implicit sewing patterns.
*Advances in Neural Information Processing Systems* 36 (2024).
- Li et al. (2024a)
Yifei Li, Hsiao-yu Chen, Egor Larionov, Nikolaos Sarafianos, Wojciech Matusik, and Tuur Stuyck. 2024a.
DiffAvatar: Simulation-Ready Garment Optimization with Differentiable Simulation.
*IEEE Conference on Computer Vision and Pattern Recognition(CVPR)* (2024).
- Li et al. (2022)
Yifei Li, Tao Du, Kui Wu, Jie Xu, and Wojciech Matusik. 2022.
Diffcloth: Differentiable cloth simulation with dry frictional contact.
*ACM Transactions on Graphics (TOG)* 42, 1 (2022), 1–20.
- Liang et al. (2019)
Junbang Liang, Ming Lin, and Vladlen Koltun. 2019.
Differentiable cloth simulation for inverse problems.
*Advances in Neural Information Processing Systems* 32 (2019).
- Lipman et al. (2008)
Yaron Lipman, David Levin, and Daniel Cohen-Or. 2008.
Green coordinates.
*ACM transactions on graphics (TOG)* 27, 3 (2008), 1–10.
- Liu et al. (2023)
Lijuan Liu, Xiangyu Xu, Zhijie Lin, Jiabin Liang, and Shuicheng Yan. 2023.
Towards garment sewing pattern reconstruction from a single image.
*ACM Transactions on Graphics (TOG)* 42, 6 (2023), 1–15.
- Loper et al. (2015)
Matthew Loper, Naureen Mahmood, Javier Romero, Gerard Pons-Moll, and Michael J. Black. 2015.
SMPL: A Skinned Multi-Person Linear Model.
*ACM Trans. Graphics (Proc. SIGGRAPH Asia)* 34, 6 (Oct. 2015), 248:1–248:16.
- Macklin et al. (2016)
Miles Macklin, Matthias Müller, and Nuttapong Chentanez. 2016.
XPBD: position-based simulation of compliant constrained dynamics. In *Proceedings of the 9th International Conference on Motion in Games*. 49–54.
- Martin et al. (2011)
Sebastian Martin, Bernhard Thomaszewski, Eitan Grinspun, and Markus Gross. 2011.
Example-based elastic materials.
In *ACM SIGGRAPH 2011 papers*. 1–8.
- Müller et al. (2007)
Matthias Müller, Bruno Heidelberger, Marcus Hennix, and John Ratcliff. 2007.
Position based dynamics.
*Journal of Visual Communication and Image Representation* 18, 2 (2007), 109–118.
- Pietroni et al. (2022)
Nico Pietroni, Corentin Dumery, Raphael Falque, Mark Liu, Teresa A Vidal-Calleja, and Olga Sorkine-Hornung. 2022.
Computational pattern making from 3D garment models.
*ACM Trans. Graph.* 41, 4 (2022), 157–1.
- Sarafianos et al. (2024)
Nikolaos Sarafianos, Tuur Stuyck, Xiaoyu Xiang, Yilei Li, Jovan Popovic, and Rakesh Ranjan. 2024.
Garment3DGen: 3D Garment Stylization and Texture Generation.
*arXiv preprint arXiv:2403.18816* (2024).
- Shewchuk (2002)
Jonathan Richard Shewchuk. 2002.
What is a Good Linear Element? Interpolation, Conditioning, and Quality Measures. In *International Meshing Roundtable Conference*.
[https://api.semanticscholar.org/CorpusID:8691914](https://api.semanticscholar.org/CorpusID:8691914)
- Stuyck (2022)
Tuur Stuyck. 2022.
*Cloth simulation for computer graphics*.
Springer Nature.
- Stuyck and Chen (2023)
Tuur Stuyck and Hsiao-yu Chen. 2023.
Diffxpbd: Differentiable position-based simulation of compliant constraint dynamics.
*Proceedings of the ACM on Computer Graphics and Interactive Techniques* 6, 3 (2023), 1–14.
- Terzopoulos et al. (1987)
Demetri Terzopoulos, John Platt, Alan Barr, and Kurt Fleischer. 1987.
Elastically deformable models.
*SIGGRAPH Comput. Graph.* 21, 4 (aug 1987), 205–214.
[https://doi.org/10.1145/37402.37427](https://doi.org/10.1145/37402.37427)
- Wang (2018)
Huamin Wang. 2018.
Rule-free sewing pattern adjustment with precision and efficiency.
*ACM Transactions on Graphics (TOG)* 37, 4 (2018), 1–13.
- Wojtan et al. (2006)
Chris Wojtan, Peter J Mucha, and Greg Turk. 2006.
Keyframe control of complex particle systems using the adjoint method. In *Proceedings of the 2006 ACM SIGGRAPH/Eurographics symposium on Computer animation*. 15–23.
- Wolff et al. (2023)
Katja Wolff, Philipp Herholz, Verena Ziegler, Frauke Link, Nico Brügel, and Olga Sorkine-Hornung. 2023.
Designing Personalized Garments with Body Movement. In *Computer Graphics Forum*, Vol. 42. Wiley Online Library, 180–194.
- Xiang et al. (2022)
Donglai Xiang, Timur Bagautdinov, Tuur Stuyck, Fabian Prada, Javier Romero, Weipeng Xu, Shunsuke Saito, Jingfan Guo, Breannan Smith, Takaaki Shiratori, et al. 2022.
Dressing avatars: Deep photorealistic appearance for physically simulated clothing.
*ACM Transactions on Graphics (TOG)* 41, 6 (2022), 1–15.
- Yang et al. (2018)
Shan Yang, Zherong Pan, Tanya Amert, Ke Wang, Licheng Yu, Tamara Berg, and Ming C. Lin. 2018.
Physics-Inspired Garment Recovery from a Single-View Image.
*ACM Trans. Graph.* 37, 5, Article 170 (nov 2018), 14 pages.
[https://doi.org/10.1145/3026479](https://doi.org/10.1145/3026479)
- Zheng et al. (2024)
Yang Zheng, Qingqing Zhao, Guandao Yang, Wang Yifan, Donglai Xiang, Florian Dubost, Dmitry Lagun, Thabo Beeler, Federico Tombari, Leonidas Guibas, and Gordon Wetzstein. 2024.
PhysAvatar: Learning the Physics of Dressed 3D Avatars from Visual Observations.
*arxiv*.