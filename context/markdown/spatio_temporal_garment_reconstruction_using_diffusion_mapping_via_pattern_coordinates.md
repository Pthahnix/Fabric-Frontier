<!-- markdownlint-disable -->
## Contents
- I Introduction
- II Related Work
  - II-A Static Garment Reconstruction.
  - II-B Dynamic Garment Reconstruction.
  - II-C Diffusion Models.
- III Garment Representation Model
  - III-A Implicit Sewing Patterns
    - Formalization
    - Training
  - III-B Extending ISP with a Diffusion Model
- IV Static Garment Reconstruction
  - IV-A Observations from Images
  - IV-B Mapping from Pixel Space to UV Space and 3D Space
    - To 3D Space
    - To UV Space
    - Training
  - IV-C Fitting
    - IV-C 1 Recovering the Rest Geometry.
    - IV-C 2 Recovering Deformed Geometry.
    - IV-C 3 Garment Refinement
- V Dynamic Garment Reconstruction
  - V-A Spatio-Temporal Diffusion
    - V-A 1 Spatial Module
    - V-A 2 Temporal Module
    - V-A 3 Training
  - V-B Test-Time Guidance for Long Sequences
    - V-B 1 Guidance in ST Normal Diffusion
    - V-B 2 Guidance in ST Mapping Diffusion
  - V-C Projection-Based Constraint for Deformation Modeling
  - V-D Refinement
- VI Experiments
  - VI-A Dataset and Evaluation Metrics
    - VI-A 1 Dataset
    - VI-A 2 Evaluation Metrics
  - VI-B Reconstruction Results
    - VI-B 1 Quantitative Comparison
    - VI-B 2 Qualitative Comparison
    - VI-B 3 Reconstructions on real-world images and videos
    - VI-B 4 Inference Time
  - VI-C Ablation Study
    - VI-C 1 Effectiveness of Fitting Strategy
    - VI-C 2 Effectiveness of Spatial Guidance
    - VI-C 3 Effectiveness of Temporal Components
    - VI-C 4 Effectiveness of Projection-Based Inpainting
  - VI-D Downstream Applications
    - Retargeting
    - Texture Editing
- VII Conclusion
- Acknowledgments
- References
- VIII Supplementary Material
  - VIII-A Additional Qualitative Results
    - Static Reconstruction
    - Dynamic Reconstruction
    - Visualization of Error
  - VIII-B Range-Null Space Decomposition and DDPM p
  - VIII-C Recovering Rest Geometry with Accumulated Partial Masks for Dynamic Reconstruction
  - VIII-D Implementation Details
    - Networks and Training
    - Inference
    - Fitting

## Abstract

Abstract Reconstructing 3D clothed humans from monocular images and videos is a fundamental problem with applications in virtual try-on, avatar creation, and mixed reality. Despite significant progress in human body recovery, accurately reconstructing garment geometry, particularly for loose-fitting clothing, remains an open challenge. We propose a unified framework for high-fidelity 3D garment reconstruction from both single images and video sequences. Our approach combines Implicit Sewing Patterns (ISP) with a generative diffusion model to learn expressive garment shape priors in 2D UV space. Leveraging these priors, we introduce a mapping model that establishes correspondences between image pixels, UV pattern coordinates, and 3D geometry, enabling accurate and detailed garment reconstruction from single images. We further extend this formulation to dynamic reconstruction by introducing a spatio-temporal diffusion scheme with test-time guidance to enforce long-range temporal consistency. We also develop analytic projection-based constraints that preserve image-aligned geometry in visible regions while enforcing coherent completion in occluded areas over time. Although trained exclusively on synthetically simulated cloth data, our method generalizes well to real-world imagery and consistently outperforms existing approaches on both tight- and loose-fitting garments. The reconstructed garments preserve fine geometric detail while exhibiting realistic dynamic motion, supporting downstream applications such as texture editing, garment retargeting, and animation.

## I Introduction

Recovering human body pose and shape, together with garment geometry, from visual input alone is a long-standing problem in computer vision with numerous applications, including fashion design, virtual try-on, 3D avatar creation, telepresence, and immersive VR/AR. In recent years, substantial progress has been made in modeling people wearing tight-fitting clothing, both in terms of body pose estimation [5, 36, 31, 70, 43] and garment shape reconstruction [11, 3, 28, 10, 51]. However, accurately modeling clothing, particularly loose-fitting garments, remains a significant challenge. Most existing approaches rely on a single unified 3D representation that jointly models the body and clothing. While such fused models can yield visually compelling results, they preclude realistic cloth simulation and limit applications such as virtual try-on.

To enable these applications, independent models for the body and garments are required. Unfortunately, garment modeling is inherently difficult due to the complex structure of clothing. Garments are thin surfaces with extremely high degrees of freedom and exhibit complex deformations driven by body motion and cloth dynamics. In addition, the diversity of garment designs and shape variations further complicates modeling and makes the acquisition of real 3D training data challenging, hindering the deployment of learning-based reconstruction methods. To mitigate these issues, several works [11, 3, 28, 6, 46] adopt pre-designed mesh templates to define garment geometry and employ linear blend skinning (LBS) [48] driven by an underlying body model to capture pose-induced deformations. While effective for tight-fitting clothing, this strategy requires predefined garment templates, limiting modeling flexibility and generalization. More critically, skinning-based methods struggle to represent loose garments that move independently and often deviate substantially from the body surface.

In prior work [41, 40], we addressed these limitations by introducing *Implicit Sewing Patterns* (ISP), representing garments as collections of 2D panels with associated 3D surfaces. A learned deformation model allows these surfaces to deviate significantly from the body shape, enabling the modeling of loose garments. While effective, this approach tends to over-smooth the geometry, particularly in occluded regions that are not directly observable from the input image.

Moreover, the aforementioned methods are primarily designed for single-image reconstruction, which is insufficient for practical applications such as character animation, motion capture, and motion analysis, where temporal consistency is essential for both visual realism and physical plausibility. Applying single-frame methods independently to each frame of a video sequence produces severe temporal artifacts, including flickering and implausible garment motion, as these approaches fail to capture the highly nonlinear dynamics of clothing. Recent video-based approaches attempt to enforce temporal consistency, but they face notable limitations: body-driven methods [55, 7] struggle to capture large-scale and time-stable garment dynamics independent of the body, while approaches that learn garment-specific deformation fields [20, 12] often over-smooth the geometry and inadequately model body-garment interactions.

In this paper, we introduce DMap, a unified diffusion-based framework for high-fidelity 3D garment reconstruction from both single images and video sequences. Our method leverages multiple diffusion processes to learn powerful garment shape priors capable of modeling complex geometries, completing occluded regions, and mapping 2D image observations to 3D and UV spaces to recover plausible garment shapes from static images. It also incorporates a spatio-temporal diffusion framework, integrating spatial priors with temporal dynamics through test-time guidance. This enables DMap to reconstruct dynamic garments with high geometric accuracy while preserving fine surface details and ensuring temporal consistency across video frames.

We proposed an early version, DMap-Static, in a conference paper [38], but it was only designed for single images. In this journal paper, we extend it to DMap-Dynamic to also handle video sequences. This is far from a trivial extension. On one hand, the framework must ensure temporal consistency for long videos despite limited GPU memory. On the other hand, it must maintain the high-fidelity of per-frame reconstruction without suffering from the over-smoothing common in dynamic models [20, 12]. To address this, our extended method builds a spatio-temporal framework that carefully leverages both spatial and temporal knowledge, together with advanced test-time guidance mechanisms.
The main contributions beyond the conference version are summarized as follows:

- •
We propose a spatio-temporal diffusion framework that explicitly decouples spatial and temporal modeling. The pre-trained spatial garment priors are reused without costly fine-tuning, while a lightweight, plug-and-play temporal module is trained to capture garment dynamics, enabling high-fidelity and temporally consistent 4D garment reconstruction.
- •
We introduce a test-time guidance strategy that enforces long-range temporal consistency under limited GPU memory by blending learned garment priors with realistic constraints, preserving both spatial detail and temporal smoothness in long video sequences.
- •
We develop analytic projection-based constraints that preserve visible garment geometry while enforcing spatio-temporal consistency in occluded regions.

We conduct extensive experiments on benchmark datasets and in-the-wild videos. Although trained exclusively on synthetically simulated cloth data, DMap generalizes well to real-world imagery and consistently outperforms existing approaches in terms of both geometric fidelity and temporal consistency.
The code and pre-trained models are publicly available at [https://github.com/kasvii/DMap](https://github.com/kasvii/DMap).

## II Related Work

### II-A Static Garment Reconstruction.

Reconstructing clothed humans from single images has progressed significantly. Learning-based methods [11, 3, 28] predict vertex displacements and deform predefined garment templates into the target pose, while optimization-based methods [6, 46] refine vertex positions to fit estimated normal maps for fine details. However, the reliance on garment templates limits the flexibility and generalization of these methods. The methods of [10, 51, 42] employ Signed Distance Functions (SDFs) to recover more diverse garment geometries. However, since SDFs represent watertight surfaces, modeling non-watertight garments requires enclosing them within thin volumes, which compromises geometric fidelity and hinders downstream refinements. In [13, 18], the SDFs are replaced by Unsigned Distance Functions (UDFs) to remove this limitation. Unfortunately, training a network to produce an accurate UDF is often more difficult than training it to yield an SDF, often resulting in inaccurate reconstructed 3D meshes with unwarranted holes. ISP [41] solves this by leveraging 2D sewing patterns and UV-based parameterization to represent garments in a more structured and interpretable manner. However, it struggles to capture large garment deformations. DMap-Static [38] builds upon ISP and combines it with a generative diffusion model to learn the possible shape distribution represented by UV positional maps, allowing the reconstruction of highly detailed garments undergoing large non-rigid deformations. However, naively applying these single-image methods to video frames without explicitly enforcing temporal consistency often leads to jittery garment motion and unnatural dynamics. This issue is exacerbated by frame-to-frame inconsistencies in the reconstructed geometry, particularly in unseen surface regions.

### II-B Dynamic Garment Reconstruction.

Garment reconstruction from videos presents unique challenges due to complex clothing dynamics.
In [44, 55], a temporal deformation field that transfers vertices of a canonical template mesh to individual video frames is learned. However, the implicit encoding of time yields only weak temporal consistency, and the reliance on a fixed template mesh limits generality. The methods in [29, 27, 53, 19, 21, 68, 66, 30] model garments using neural implicit functions, producing visually compelling reconstructions across a wide range of garment types. However, these methods fuse the body and clothes into a single entity, which limits their usefulness for downstream tasks such as garment editing, simulation, and virtual try-on. Recent works, such as REC-MV [55] and D^3-Human [7], introduce separate representations for the garment and body. Unfortunately, their body-driven deformation strategies struggle to capture large-scale and time-stable garment displacements away from the body, which are common in loose-fitting clothing. Moreover, temporal consistency is only implicitly conveyed through body motion, which provides weak constraints on garment dynamics. To improve on this, ReLoo [20] introduces virtual bones that can be freely transferred to capture dynamic deformations and perform global sequence optimization. This yields smooth garment motion, but sacrifices per-frame accuracy and coarsens the reconstruction. NGD [12] also separates the body and garment and models garment deformation using a neural Jacobian field. While it can capture large dynamic motions of loose garments, it does not adequately model physical interactions and mutual constraints between the body and the garment. Overall, existing methods still struggle to provide the required balance between temporal consistency and spatial reconstruction accuracy under real-world physical interactions, which is precisely what our DMap is designed to address.

### II-C Diffusion Models.

Diffusion models [25, 63] learn complex data distributions through a forward-reverse noise process and have achieved remarkable success in generating high-quality and diverse images. Building on this foundation, recent works [15, 62, 26, 49, 23] extend image diffusion models to the temporal domain, enabling video processing and synthesis. In the context of garment reconstruction, diffusion-based shape priors have recently been explored [22, 39], leveraging UV-space parameterizations to model garment geometry. However, these methods require point clouds as 3D garment measurements and do not account for body-garment interaction during reconstruction. In contrast, our method reconstructs 3D garments directly from monocular 2D images while explicitly modeling both the garment and the underlying body. Furthermore, we extend spatial diffusion with a temporal module capable of processing video sequences. Instead of directly adopting video diffusion methods, which are typically restricted to short clips due to memory limitations, we introduce a testing-time guidance mechanism that incorporates realistic constraints, including long-range temporal consistency and accurate 2D-3D alignment, so that our approach overcomes the limitations of existing video diffusion methods and delivers high-fidelity and temporally consistent garment reconstructions across long video sequences.

Figure: Figure 2: Pipeline. Given an image of a clothed person, we first estimate the front normal $\mathbf{n}_{F}$ of the target garment, and the SMPL body model which is used to render the body part segmentation ($\mathbf{s}_{F}$, $\mathbf{s}_{B}$) and depth ($\mathbf{d}_{F}^{b}$, $\mathbf{d}_{B}^{b}$) images. The back normal $\mathbf{n}_{B}$ of the garment is estimated subsequently by the diffusion model $\boldsymbol{\epsilon}_{\theta}^{n}$. We then predict the UV-coordinate ($\mathbf{c}_{F}$, $\mathbf{c}_{B}$) and the depth ($\mathbf{d}_{F}^{g}$, $\mathbf{d}_{B}^{g}$) images from the garment normal and body estimations with the mapping model $\boldsymbol{\epsilon}_{\theta}^{m}$. The incomplete UV positional map $\tilde{\mathcal{U}}$ is produced from them using the camera backprojection. Finally, we fit $\tilde{\mathcal{U}}$ to DISP to recover the complete UV positional map $\hat{\mathcal{U}}$ and the corresponding garment mesh $\mathbf{g}$, which is further improved by the refinement.
Refer to caption: 2602.24043v1/x1.png

## III Garment Representation Model

In this section, we define our garment representation model, DISP. It defines the reconstruction prior we use when reconstructing garment shapes from either single images or whole video sequences, as discussed in Sec. [IV](#S4) and [V](#S5).
DISP relies on Implicit Sewing Pattern (ISP) [41] to model the garment rest geometry. ISP uses UV positional maps to model the geometry of different garments but is limited by only producing a single UV map for a specific garment, which is not enough to represent the many possible answers. To address this issue, we extend ISP into DISP by incorporating a diffusion model to capture the complex garment shapes caused by body motion. It is depicted by the gray network in Fig. [2](#S2.F2). In the remainder of this section, we first introduce ISP and then describe how we incorporate a diffusion model into it.

### III-A Implicit Sewing Patterns

##### Formalization

Implicit Sewing Patterns (ISP) [41] is a garment model based on the sewing patterns used in the fashion industry to design and manufacture clothes. A sewing pattern is made of several 2D panels along with stitch information for assembling them together. The 2D panels and the stitching are implicitly modeled using a 2D signed distance field (SDF) and a 2D label field, respectively. For a specific garment, its corresponding latent code $\mathbf{z}$, and a point $\mathbf{u}$ in the 2D UV space $\Omega=[-1,1]^{2}$, the ISP model outputs the signed distance $s$ to the panel boundary and a label $l$ using a fully connected network $\mathcal{I}_{\mathbf{\Theta}}$ as

$$ $(s,l)=\mathcal{I}_{\mathbf{\Theta}}(\mathbf{u},\mathbf{z})\;.$ (1) $$

The zero crossing of the SDF defines the shape of the panel, with $s<0$ indicating that $\mathbf{u}$ is within the panel and $s>0$ indicating that $\mathbf{u}$ is outside the panel. The label $l$ encodes the stitch information, instructing which panel boundaries should be stitched together. To map the 2D sewing patterns to 3D surfaces, a UV parameterization function $\mathcal{A}_{\Phi}$ is learned to perform the 2D-to-3D mapping

$$ $\mathbf{X}=\mathcal{A}_{\Phi}(\mathbf{u},\mathbf{z})\;,$ (2) $$

where $\mathbf{X}\in\mathbb{R}^{3}$ represents the 3D position of $\mathbf{u}$. In essence, ISP registers different garments onto a unified UV space and establishes the mapping functions between points in UV space and the 3D garment surfaces. The shape of SDF’s 0-crossing defines the geometry of garment in its rest state. As ISP is a differentiable representation, we can easily fit a latent code $\mathbf{z}$ to arbitrary masks or contours of the panels to recover the corresponding garment geometry.

##### Training

Training ISP requires the 2D sewing patterns of rest-state 3D garments. However, they are not available in most garment datasets, e.g. CLOTH3D [2]. Following the garment flattening strategy of [40, 54], we cut the garment mesh of CLOTH3D into front and back surfaces according to predefined cutting rules and then flatten them into 2D panels by minimizing an as-rigid-as-possible energy [45].
For each garment in the dataset, a front and a back panel are generated as its sewing pattern. By using the paired 2D sewing patterns and their 3D meshes, we learn the weights of the ISP model $(\mathcal{I}_{\mathbf{\Theta}},\mathcal{A}_{\Phi})$ with the training procedure of [41].

### III-B Extending ISP with a Diffusion Model

For a specific garment, the UV parameterization function of ISP only produces a single UV positional map $\mathcal{U}_{r}$ to model its 3D shape in the rest state

$$ $\mathcal{U}_{r}[u,v]=\begin{cases}\mathcal{A}_{\Phi}(\mathbf{u},\mathbf{z}),&\mbox{if }s_{\mathbf{u}}\leq 0\\ \varnothing,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;,$ (3) $$

where $\mathbf{u}=(u,v)$, $s_{\mathbf{u}}$ is the SDF value of $\mathbf{u}$, $[\cdot,\cdot]$ denotes the standard array addressing and $\varnothing=(-1,-1,-1)$ indicates the region outside the panel.
When dressed on the body, the garment can have various deformations due to the motion of the underlying body, which is not able to be modeled by ISP solely. Inspired by [22, 39], we incorporate a diffusion model into ISP to capture these possible deformations by generating plausible UV maps.

Given the deformed garments worn on the body whose rest states are modeled by ISP as Eq. [3](#S3.E3), we write the corresponding UV maps

$$ $\mathcal{U}[u,v]=\begin{cases}\mathbf{V},&\mbox{if }s_{\mathbf{u}}\leq 0\\ \varnothing,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;,$ (4) $$

where $\mathbf{V}\in\mathbb{R}^{3}$ is the corresponding position on the deformed mesh surface for the UV point $\mathbf{u}=(u,v)$. Each $\mathcal{U}$ represents a specific deformed shape for a particular garment. We use a diffusion model [24] $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$ to learn the distribution of plausible deformations represented by $\mathcal{U}$.

For each garment sample, we generate its UV map $\mathcal{U}$ according to Eq. [4](#S3.E4), along with a panel mask $\mathcal{M}$ as

$$ $\mathcal{M}[u,v]=\begin{cases}1,&\mbox{if }s_{\mathbf{u}}\leq 0\\ 0,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;.$ (5) $$

$\mathcal{M}$ depicts the panel shape, which itself encodes the 3D geometry of the canonical rest garment. We concatenate $\mathcal{U}$ and $\mathcal{M}$ along the channel dimension to form the training samples and train the network $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$ on them.
After training, the diffusion model and ISP together form the garment model DISP.

## IV Static Garment Reconstruction

Monocular images provide only partial 2D observations of visible regions. To complete the missing information in occluded areas and enable globally consistent reconstruction, we propose DMap-Static, a framework combining three diffusion schemes trained to
(1) learn a garment shape prior,
(2) infer image information for occluded garment regions, and
(3) map the 2D image to both UV and 3D spaces to recover plausible 3D shapes align with the learned prior.

### IV-A Observations from Images

Recent advances in image segmentation [35, 57], normal estimation [1, 33] and human mesh recovery [17, 64] can be used to extract accurate observations from an image of a clothed person. In this manner,
we first segment the target garment using [35, 37] and estimate its normals $\mathbf{n}_{F}$ using [33]. To model the body underneath, we use SMPL [48], which relies on two sets of parameters $(\beta,\theta)$ to describe the body shape and pose respectively. The SMPL parameters are estimated from the image by [17, 61] to infer the 3D body shape, which are then used to render front and back body part segmentations $\mathbf{s}_{F}$ and $\mathbf{s}_{B}$, along with front and back depth images $\mathbf{d}_{F}^{b}$ and $\mathbf{d}_{B}^{b}$, as shown in the top left of Fig. [2](#S2.F2).

To estimate the invisible normals, typically in the back, $\mathbf{n}_{B}$ as shown in Fig. [2](#S2.F2), we use the estimated normal $\mathbf{n}_{F}$ to guide a conditional diffusion model $\boldsymbol{\epsilon}_{\theta}^{n}$. The denoising process of $\boldsymbol{\epsilon}_{\theta}^{n}$ is conditioned on the visible normals $\mathbf{n}_{F}$, the front and back segmentation images $\mathbf{s}=(\mathbf{s}_{F},\mathbf{s}_{B})$, the body depth maps $\mathbf{d}^{b}=(\mathbf{d}_{F}^{b},\mathbf{d}_{B}^{b})$. It is learned by minimizing the loss

$$ $\mathcal{L}=\mathbb{E}_{t,\mathbf{N}_{B},\boldsymbol{\epsilon}}\|\boldsymbol{\epsilon}-\boldsymbol{\epsilon}_{\theta}^{n}\left(\sqrt{\bar{\alpha}_{t}}\mathbf{n}_{B}+\sqrt{1-\bar{\alpha}_{t}}\boldsymbol{\epsilon},\mathbf{n}_{F},\mathbf{s},\mathbf{d}^{b},t\right)\|^{2}\;.$ (6) $$

The conditioning images of the garment and body provide information to generate plausible normals for the back of the garment. As will be shown in our experiments, the back normal estimation $\mathbf{n}_{B}$ provides additional constraints to regularize the garment fitting process, which improves the fidelity of the reconstruction.

### IV-B Mapping from Pixel Space to UV Space and 3D Space

Figure: Figure 3: Mapping between pixel, 3D, and UV spaces. The pixel $(x,y)$ is mapped to $(X,Y,Z)$ in the 3D space using the estimated depth $d$ and the camera backprojection $P^{-1}$, and to $(u,v)$ in the UV space using the estimated UV coordinates $(u,v,\sigma)$. The dash line indicates that $(X,Y,Z)$ and $(u,v)$ are connected indirectly through $(x,y)$.
Refer to caption: 2602.24043v1/x2.png

The normal estimations provide observations in pixel space, while the garment model DISP is learned in the UV space of the garment panels, and the garment surface resides in the 3D space. To reconstruct 3D garments using DISP, it is thus necessary to connect these three different spaces. To this end, we introduce a mapping function that translates image observations from the pixel space to both the UV space and the 3D space, as illustrated by Fig. [3](#S4.F3).

##### To 3D Space

Since the depth and surface normal are closely related in terms of 3D geometry, we estimate the garment depth image $\mathbf{d}^{g}$ from normal estimations $\mathbf{n}=(\mathbf{n}_{F},\mathbf{n}_{B})$ conditioned on the body depth $\mathbf{d}^{b}$. For the foreground pixel $(x,y)$, its absolute depth value is $d=\mathbf{d}^{g}[x,y]$. By leveraging the camera projection $P$, we can have the 3D coordinate $(X,Y,Z)$ for each pixel

$$ $(X,Y,Z)=P^{-1}(x,y,d)\;,$ (7) $$

where $P^{-1}$ denotes the camera backprojection. Through Eq. [7](#S4.E7), we establish the mapping from the pixel space to the 3D space.

##### To UV Space

Given the normal estimation $\mathbf{n}$, we train a network $\boldsymbol{\epsilon}_{\theta}^{m}$ to predict a UV-coordinate image $\mathbf{c}$ conditioned on the body part segmentation $\mathbf{s}$. The pixel value of $\mathbf{c}$ is

$$ $\mathbf{c}[x,y]=(u,v,\sigma)\;,$ (8) $$

where $(u,v)$ is the predicted coordinate on the UV space of the panel for pixel $(x,y)$, $\sigma$ indicates whether it belongs to the front ($\sigma>0$) or the back ($\sigma<0$) panel. Through Eq. [8](#S4.E8), we establish the mapping from the pixel space to the UV space.

By assembling the results of UV and 3D mapping of Eq. [8](#S4.E8) and Eq. [7](#S4.E7), we can get a UV map $\tilde{\mathcal{U}}$, where

$$ $\tilde{\mathcal{U}}[u,v]=P^{-1}(x,y,d)=(X,Y,Z)\;.$ (9) $$

For the positions on $\tilde{\mathcal{U}}$ without projected points, we simply set their values to $\varnothing$. We also compute a mask $\tilde{\mathcal{M}}$ with
$\tilde{\mathcal{M}}[u,v]=1$ at where a pixel is projected, and $\tilde{\mathcal{M}}[u,v]=0$ otherwise. Due to occlusions, both $\tilde{\mathcal{U}}$ and $\tilde{\mathcal{M}}$ are incomplete. In the next section, we will complete them by fitting to the priors encoded in DISP.

##### Training

We learn the mapping function in an image-to-image translation fashion with a conditional diffusion model $\boldsymbol{\epsilon}_{\theta}^{m}$. For the normal estimation $\mathbf{n}_{F}$ and $\mathbf{n}_{B}$, $\boldsymbol{\epsilon}_{\theta}^{m}$ is trained to predict their UV-coordinate image $\mathbf{c}_{F}$ and $\mathbf{c}_{B}$, and depth images $\mathbf{d}_{F}^{g}$ and $\mathbf{d}_{B}^{g}$ jointly.
The denoising process of $\boldsymbol{\epsilon}_{\theta}^{m}$ is conditioned on the estimated normals of the front and the back $\mathbf{n}=(\mathbf{n}_{F},\mathbf{n}_{B})$, the segmentation images $\mathbf{s}=(\mathbf{s}_{F},\mathbf{s}_{B})$, the body depth maps $\mathbf{d}^{b}=(\mathbf{d}_{F}^{b},\mathbf{d}_{B}^{b})$, and is learned by minimizing the loss

$$ $\mathcal{L}=\mathbb{E}_{t,\mathbf{m}_{0},\boldsymbol{\epsilon}}\|\boldsymbol{\epsilon}-\boldsymbol{\epsilon}_{\theta}^{m}\left(\sqrt{\bar{\alpha}_{t}}\mathbf{m}_{0}+\sqrt{1-\bar{\alpha}_{t}}\boldsymbol{\epsilon},\mathbf{n},\mathbf{s},\mathbf{d}^{b},t\right)\|^{2}\;,$ (10) $$

where $\mathbf{m}_{0}=[\mathbf{c}_{F},\mathbf{c}_{B},\mathbf{d}_{F}^{g},\mathbf{d}_{B}^{g}]$. After the training, we assemble the results for both the front and the back to produce the UV map $\tilde{\mathcal{U}}$ and the mask $\tilde{\mathcal{M}}$. Compared with only using the front result, this provides more observations and constraints for the fitting, resulting in a reconstruction with higher quality for both the visible and invisible parts.

### IV-C Fitting

The incomplete panel mask $\tilde{\mathcal{M}}$ and the incomplete UV map $\tilde{\mathcal{U}}$ provides partial information of the garment geometry and deformation, respectively. To recover a complete garment from them, we leverage the prior of DISP. We first recover the complete panel mask to recover the garment geometry in its rest shape, and then recover the complete UV map for the deformation. To remedy the synthetic-to-real domain gap and further improve the reconstruction accuracy, we rely on a post-optimization step to align the garment with image observations.

Figure: Figure 4: Recovering garment rest geometry. Given (a) the incomplete panel mask $\tilde{\mathcal{M}}$, we fit (b) the complete panel mask $\mathcal{M}$ by Eq. [11](#S4.E11). (c) shows the overlay of $\tilde{\mathcal{M}}$ in gray and $\mathcal{M}$ in white. (d) is the corresponding rest-state garment mesh $\bar{\mathbf{g}}$ for (b).
Refer to caption: 2602.24043v1/x3.png

#### IV-C 1 Recovering the Rest Geometry.

To recover the garment rest geometry represented by the 2D panel shape from $\tilde{\mathcal{M}}$, we optimize the latent code $\mathbf{z}$ of Eq. [1](#S3.E1) so that its corresponding patterns match $\tilde{\mathcal{M}}$ as well as possible. The optimization objective is

$$ $\mathcal{L}(\mathbf{z})=\sum\limits_{\scalebox{0.7}{$\mathbf{u}\in\mathcal{M}_{+}$}}ReLU(s_{\mathbf{u}}(\mathbf{z}))-\lambda_{area}\sum\limits_{\scalebox{0.7}{$\mathbf{u}\in\Omega$}}s_{\mathbf{u}}(\mathbf{z})+\lambda_{\mathbf{z}}||\mathbf{z}||_{2}\;,$ (11) $$

where $\mathcal{M}_{+}=\{\mathbf{u}|\tilde{\mathcal{M}}_{\mathbf{u}}=1,\mathbf{u}\in\Omega\}$, $s_{\mathbf{u}}(\mathbf{z})$ is the SDF value of $\mathbf{u}$ computed by ISP, and $\lambda_{area}$ and $\lambda_{\mathbf{z}}$ are the weighting constants. The first item in Eq. [11](#S4.E11) ensures that the projected UV points are within the panel, while the second one penalizes large panel area to make the panel contours surround the non-zero points of $\tilde{\mathcal{M}}$ as closely as possible. This optimization produces an optimal latent code $\mathbf{z}^{*}$ that we can use to infer a complete panel mask $\mathcal{M}$ and a rest-state garment mesh $\bar{\mathbf{g}}$ as shown in Fig. [4](#S4.F4).

#### IV-C 2 Recovering Deformed Geometry.

The diffusion model $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$ of DISP learns the distribution of plausible deformations represented by UV maps. To recover the full UV map $\mathcal{U}$, we use the partial UV map $\tilde{\mathcal{U}}$ and the recovered panel mask $\mathcal{M}$ as the manifold guidance [8, 9] in the reverse diffusion process of $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$:

$$ $\displaystyle\nabla_{\mathbf{x}_{t}}\log p(\mathbf{x}_{t}|\tilde{\mathcal{U}},\tilde{\mathcal{M}},\mathcal{M})$ $\displaystyle\simeq-\frac{\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t)}{\sigma_{t}}-\rho\nabla_{\mathbf{x}_{t}}\mathcal{L}(\hat{\mathbf{x}}_{0},\tilde{\mathcal{U}},\tilde{\mathcal{M}},\mathcal{M})\;,$ $\displaystyle\hat{\mathbf{x}}_{0}$ $\displaystyle=\frac{1}{\sqrt{\bar{\alpha}_{t}}}\mathbf{x}_{t}-\sqrt{\frac{1-\bar{\alpha}_{t}}{\bar{\alpha}_{t}}}\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t)\;\;,$ (12) $$

where $\rho$ is the guidance step size. $\mathcal{L}$ is the function that measures the difference between the generated and the given UV maps and panel masks

$$ $\mathcal{L}(\hat{\mathbf{x}}_{0},\tilde{\mathcal{U}},\tilde{\mathcal{M}},\mathcal{M})=\lVert\tilde{\mathcal{M}}*(\hat{\mathcal{U}}-\tilde{\mathcal{U}})\rVert_{2}+\lVert\hat{\mathcal{M}}-\mathcal{M}\rVert_{1}\;,$ (13) $$

where $\hat{\mathbf{x}}_{0}=[\hat{\mathcal{U}},\hat{\mathcal{M}}]$, $\hat{\mathcal{U}}$ and $\hat{\mathcal{M}}$ refer to the generated UV map and panel mask respectively, and $*$ denotes the element-wise multiplication. With the generated UV map $\hat{\mathcal{U}}$, we update the vertices of $\bar{\mathbf{g}}$ to get the recovered mesh $\mathbf{g}$ as shown in the bottom-left of Fig. [2](#S2.F2).

#### IV-C 3 Garment Refinement

As the diffusion model learns the shape distribution from the garment simulation data which is generated with limited materials, external forces, and body motions, when handling in-the-wild images that are out-of-distribution, it can produce inaccurate UV maps by Eq. [12](#S4.E12) and result in inaccurate garment mesh that does not align with the images. To further improve the reconstruction accuracy, we refine the recovered mesh $\mathbf{g}$ in Sec. [IV-C2](#S4.SS3.SSS2) by optimizing its vertex positions to align it with the image observations. The loss function we use is

$$ $\mathcal{L}\!=\!\lambda_{m}\mathcal{L}_{mask}\!+\!\lambda_{d}\mathcal{L}_{depth}\!+\!\lambda_{n}\mathcal{L}_{normal}\!+\!\lambda_{u}\mathcal{L}_{uv}+\lambda_{p}\mathcal{L}_{phys},$ (14) $$

where $\lambda_{m}$, $\lambda_{d}$, $\lambda_{n}$, $\lambda_{u}$ and $\lambda_{p}$ are scalar weights. $\mathcal{L}_{mask}$, $\mathcal{L}_{depth}$ and $\mathcal{L}_{normal}$ penalize the difference between the rendered front and back masks, depth and normal of the mesh $\mathbf{g}$ and their corresponding estimation, respectively. $\mathcal{L}_{uv}$ ensures the corresponding UV map $\hat{\mathcal{U}}$ of $\mathbf{g}$ aligns with the partial UV observations $\tilde{\mathcal{U}}$ by

$$ $\mathcal{L}_{uv}=\lVert\tilde{\mathcal{M}}*(\hat{\mathcal{U}}-\tilde{\mathcal{U}})\rVert_{2}.$ (15) $$

$\mathcal{L}_{phys}$ contains a set of physics-based mesh regularization [52, 59] computed by using the recovered rest-state garment $\bar{\mathbf{g}}$ as the reference

$$ $\mathcal{L}_{phys}=\mathcal{L}_{strain}+\mathcal{L}_{bend}+\mathcal{L}_{gravity}+\mathcal{L}_{collision},$ (16) $$

where $\mathcal{L}_{strain}$ is the membrane strain energy caused by the deformation, $\mathcal{L}_{bend}$ is the bending energy resulting from the folding of adjacent faces, $\mathcal{L}_{gravity}$ is the gravitational potential energy and $\mathcal{L}_{collision}$ is the penalty for body garment collision.

However, directly optimizing the vertex positions using Eq. [14](#S4.E14) will lead to a suboptimal solution, as vertices are not strongly coupled.
Inspired by [65, 16], we introduce a multi-layer perceptron (MLP) for the optimization to update vertices by a neural displacement field. To be specific, we initialize an MLP network $f$ with learnable parameters $\phi$. Given the vertex $V$ of mesh $\mathbf{g}$ and its canonical position $\bar{V}$ on $\bar{\mathbf{g}}$, the network $f_{\phi}$ predicts its displacement by

$$ $\Delta V=f_{\phi}(V,\bar{V}).$ (17) $$

We use the updated vertex position $V+\Delta V$ to compute the loss of Eq. [14](#S4.E14) and compute the gradient with respect to $\phi$ for its learning. Since neural networks tend to learn low-frequency functions [56], the result after this step is a bit smooth. To further recover fine surface details, we perform an additional refinement step by directly optimizing the garment mesh vertices with Eq. [14](#S4.E14).

Figure: Figure 5: Processing a video sequence. Given a set of images with extracted body segmentations $\mathbf{S}$, body depths $\mathbf{D}^{b}$, and garment normals $\mathbf{N}^{F}$, our method produces a sequence of garment meshes $\mathbf{G}$ in three steps. First, the back-view normals $\mathbf{N}^{B}$ of the garment are inferred. By design, our method ensures these predictions are temporally consistent. Second, a mapping network estimates the 2D/3D positions of each pixel, where the 2D positions $(\mathbf{C}_{F},\mathbf{C}_{B})$ are in a reference pattern space, and the 3D positions are represented as depth maps $(\mathbf{D}_{F}^{g},\mathbf{D}_{B}^{g})$. We introduce novel guidance on this generation to match normal estimations and prevent intersection with the body. Finally, this mapping is unwrapped into a partial 2D pattern $\tilde{\mathbf{U}}$ where pixel value encodes the 3D position, and our temporal inpainting diffusion completes these partial observations into a full garment sequence while ensuring the partial constraints are respected.
Refer to caption: 2602.24043v1/x4.png

## V Dynamic Garment Reconstruction

While DMap-Static from Sec. [IV](#S4) delivers good results from single images, due to the stochastic nature of diffusion, there is no reason for results generated independently in consecutive video frames to be consistent. Thus, frame-by-frame reconstruction does no yield realistic animations. To remedy this, we extend it into DMap-Dynamic that enforces consistency using a spatio-temporal diffusion framework, as shown in Fig. [5](#S4.F5). The main difficulty in doing so is that spatio-temporal diffusion typically requires large amounts of memory. Thus, only short video snippets can be handled at any one time. We address this by building temporal diffusion modules with across- and within-subsequence guidance to enforce temporal consistency beyond short clips. We also incorporate a novel analytic projection-based constraint that faithfully preserves the visible garment geometry while enforcing spatial and temporal consistency in occluded garment areas. We discuss these below.

Figure: Figure 6: Architecture of the spatio-temporal diffusion model. The model decouples spatial and temporal knowledge into separate modules: the spatial module (red) captures per-frame spatial structure, while the temporal module (blue) models per-pixel motion across frames. Together, these components form the foundation for high-fidelity and temporally smooth 4D garment reconstruction.
Refer to caption: 2602.24043v1/x5.png

### V-A Spatio-Temporal Diffusion

To generate a temporally consistent reconstruction over the entire video sequence, we introduce a spatio-temporal diffusion model for sequential generation of back-view normal map and 2D-to-3D mapping. As shown in Fig. [6](#S5.F6), the model decouples spatial and temporal modeling through an alternating design: the spatial module captures per-frame geometric structure, while the temporal module estimates pixel-wise motion over time. Together, they provide a unified basis for high-fidelity and temporally coherent 4D garment reconstruction.

#### V-A 1 Spatial Module

The spatial module inherits both the architecture and weights of DMap-Static, enabling us to reuse pre-trained spatial priors without the need for costly fine-tuning. As illustrated on the left of Fig. [6](#S5.F6), the module consists of convolutional and self-attention layers. Specifically, it computes self-attention across pixels within individual frames by processing a tensor of shape $\mathbb{R}^{B^{\prime}\times H\times W\times C}$, where the effective batch size $B^{\prime}=B\times T$ combines the original batch size $B$ and the number of frames $T$. This design makes the module focuses exclusively on extracting spatial structures.

#### V-A 2 Temporal Module

To extend the model’s capability to the temporal domain, we design a plug-and-play temporal module that is seamlessly inserted after each spatial module. This interleaved architecture ensures that the network captures temporal evolution hierarchically alongside spatial features. Distinct from the spatial module, the temporal module replaces convolutional layers with linear layers and computes self-attention across different frames rather than spatial positions, as shown on the right of Fig. [6](#S5.F6). In practice, this is achieved by reshaping the spatial output into a tensor of shape $\mathbb{R}^{B^{\prime\prime}\times T\times C}$, where $B^{\prime\prime}=B\times H\times W$ represents the flattened spatial dimensions.
This design enables the network to capture temporal dependencies and enforce motion consistency throughout the input sequence.

#### V-A 3 Training

Our spatio-temporal diffusion is the framework for sequential back-view normal estimation and 2D-to-3D mapping. During training, we decouple spatial and temporal learning. First, the spatial module is pre-trained on single frames in DMap-Static, allowing it to directly inherit the learned spatial priors. Then, we freeze the spatial module and train only the temporal module, allowing it to focus on learning temporal motion priors. The training objective minimizes the noise prediction error

$$ $\mathcal{L}_{ST}=\mathbb{E}_{\boldsymbol{\epsilon},t}\left[||\boldsymbol{\epsilon}-\boldsymbol{\epsilon}_{\boldsymbol{\theta}}(\mathbf{x}_{t},\mathbf{c},t)||^{2}_{2}\right]\;,$ (18) $$

where $\boldsymbol{\epsilon}\sim\mathcal{N}(\mathbf{0},\mathbf{I})$ denotes Gaussian noise and $t$ is the diffusion timestep. The conditioning input $\mathbf{c}$ is task-specific.

For spatio-temporal (ST) normal diffusion $\boldsymbol{\epsilon}_{\boldsymbol{\theta}}^{n}$, we set

$$ $\mathbf{c}_{n}=\left[\mathbf{N}^{F},\mathbf{S},\mathbf{D}^{b}\right],$ $$

where $\mathbf{N}^{F}=\{\mathbf{n}_{F}^{t}\}_{t=1}^{T}$ denotes the sequence of front-view garment normal maps, $\mathbf{S}=(\mathbf{S}_{F},\mathbf{S}_{B})$ represents the front and back body part segmentations with $\mathbf{S}_{F/B}=\{\mathbf{s}_{F/B}^{t}\}_{t=1}^{T}$, and $\mathbf{D}^{b}=(\mathbf{D}^{b}_{F},\mathbf{D}^{b}_{B})$ are the corresponding body depth images with $\mathbf{D}^{b}_{F/B}=\{\mathbf{d}_{F/B}^{b,t}\}_{t=1}^{T}$. The model predicts the back-view normal sequence $\mathbf{N}^{B}=\{\mathbf{n}_{B}^{t}\}_{t=1}^{T}$.

For ST mapping diffusion $\boldsymbol{\epsilon}_{\boldsymbol{\theta}}^{m}$, the conditioning input is

$$ $\mathbf{c}_{m}=\left[\mathbf{N}^{F},\mathbf{N}^{B},\mathbf{S},\mathbf{D}^{b}\right],$ $$

from which the model jointly predicts the garment UV-coordinate images $\mathbf{C}=(\mathbf{C}_{F},\mathbf{C}_{B})$ and the garment depth images $\mathbf{D}^{g}=(\mathbf{D}_{F}^{g},\mathbf{D}_{B}^{g})$ as shown in the top-right of Fig. [5](#S4.F5).

### V-B Test-Time Guidance for Long Sequences

#### V-B 1 Guidance in ST Normal Diffusion

ST normal diffusion aims to estimate the sequence of back-view normal maps $\mathbf{N}^{B}$, which is particularly challenging. The back view is entirely unobserved in monocular videos, requiring the model to generate geometry that is not only spatially consistent with the visible front-view normals, but also temporally coherent over long video sequences.
In practice, processing an entire video sequence at once is computationally infeasible. We therefore divide long videos into shorter subsequences, which inevitably introduces temporal discontinuities at subsequence boundaries. To address this issue, we introduce training-free temporal guidance during inference, which enforces temporal consistency both across adjacent subsequences and within each subsequence. This guidance enables stable and smooth back-view normal estimation over long video sequences without modifying the trained diffusion model.
For across-subsequence consistency, we enforce the overlapping regions between consecutive subsequences to be identical at each denoising time step

$$ $\mathcal{G}_{across}=\left|\left|\mathbf{N}^{\prime B}_{(T-N+1):T}-\hat{\mathbf{N}}^{B}_{1:N}\right|\right|^{2}_{2}\;,$ (19) $$

where $N$ is the overlapping length between adjacent subsequences, $\mathbf{N}^{\prime B}$ denotes the back-view normal maps generated for the previous clip, and $\hat{\mathbf{N}}^{B}=\hat{\mathbf{x}}_{0|t}$ denotes the posterior mean (*i.e., *the estimated clean back-view normal maps) for current clip at time step $t$.
In addition, to enforce consistency within each subsequence, we introduce velocity and acceleration losses to transfer the temporal information from the overlapping part $\hat{\mathbf{N}}^{B}_{1:N}$ to the remaining part $\hat{\mathbf{N}}^{B}_{N+1:T}$. They are given by

$$ $\mathcal{G}_{within}=\mathcal{G}_{vel}+\mathcal{G}_{acc},$ (20) $$

$$ $\mathcal{G}_{{vel}}=\frac{1}{T-1}\sum_{f=2}^{T}\left\|\hat{\mathbf{N}}^{B}_{f}-\hat{\mathbf{N}}^{B}_{f-1}\right\|_{2}^{2},$ (21) $$

$$ $\mathcal{G}_{{acc}}=\frac{1}{T-2}\sum_{f=3}^{T}\left\|\hat{\mathbf{N}}^{B}_{f}-2\hat{\mathbf{N}}^{B}_{f-1}+\hat{\mathbf{N}}^{B}_{f-2}\right\|_{2}^{2}.$ (22) $$

By using $\mathcal{G}_{temporal}=\lambda_{a}\mathcal{G}_{across}+\lambda_{w}\mathcal{G}_{within}$ as the manifold guidance [8], the sample $\mathbf{x}_{t-1}$ is updated toward improved temporal consistency

$$ $\mathbf{x}_{t-1}=\underbrace{\text{DDPM}(\mathbf{x}_{t},\boldsymbol{\epsilon}^{n}_{\boldsymbol{\theta}}(\mathbf{x}_{t},\mathbf{c}_{n},t))}_{\text{sampling}}-\underbrace{\gamma\nabla_{\mathbf{x}_{t}}\mathcal{G}_{temporal}}_{\text{temporal guidance}},$ (23) $$

where $\lambda_{a}$ and $\lambda_{w}$ are weighting coefficients, and $\gamma$ is the guidance strength parameter.

#### V-B 2 Guidance in ST Mapping Diffusion

During the transformation from 2D image space to UV and 3D spaces in ST mapping diffusion, it is crucial to both preserve spatial details and enforce temporal consistency across frames. To this end, we introduce multiple spatial and temporal guidance terms during denoising:

- •
Depth-to-normal guidance to align geometric details. We derive a map of normals $\tilde{\mathbf{N}}$ from the posterior mean depth map $\hat{\mathbf{D}}^{g}$ estimated at time step $t$, by computing its partial derivatives with respect to the $x$ and $y$ directions. We compute it as
$\tilde{\mathbf{N}}(x,y)=\frac{(-\mathbf{D}_{x},-\mathbf{D}_{y},1)^{\top}}{\sqrt{\mathbf{D}_{x}^{2}+\mathbf{D}_{y}^{2}+1}},$
(24)
$\mathbf{D}_{x}=s_{x}\frac{\partial\hat{\mathbf{D}}^{g}}{\partial x}\;,\quad\mathbf{D}_{y}=s_{y}\frac{\partial\hat{\mathbf{D}}^{g}}{\partial y}\;,$
(25)
where $s_{x}$ and $s_{y}$ are factors that match the gradients to the depth scale. We take depth-to-normal guidance loss to be $\mathcal{G}_{D2N}=1-cos(\mathbf{N},\tilde{\mathbf{N}})$, where $\mathbf{N}$ is the input normal map. Minimizing this term encourages $\tilde{\mathbf{N}}$ to match $\mathbf{N}$.
- •
Interpenetration-aware guidance to encourage the garment surface to remain outside the body. We define the guidance loss as
$\mathcal{G}_{inter}=\text{ReLU}\left(\mathbf{D}^{b}_{F}-\hat{\mathbf{D}}^{g}_{F}\right)+\text{ReLU}\left(\hat{\mathbf{D}}^{g}_{B}-\mathbf{D}^{b}_{B}\right)\;,$
(26)
where $\mathbf{D}^{b}_{F}$ and $\mathbf{D}^{b}_{B}$ are the front and back body depths, respectively. $\hat{\mathbf{D}}^{g}_{F}$ and $\hat{\mathbf{D}}^{g}_{B}$ are the front and back garment depths derived by the posterior mean at time step $t$. Minimizing this term prevents body-garment interpenetrations.
- •
Temporal-consistency guidance to enforce both across- and within-subsequence consistency.
We use the same guidance function $\mathcal{G}_{temporal}$ as in Sec. [V-B1](#S5.SS2.SSS1).

The gradients of these guidance terms are injected into the denoising process by writing

$$ $\displaystyle\mathbf{x}_{t-1}$ $\displaystyle=\text{DDPM}\big(\mathbf{x}_{t},\,\boldsymbol{\epsilon}^{m}_{\boldsymbol{\theta}}(\mathbf{x}_{t},\mathbf{c}_{m},t)\big)-\gamma_{n}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{D2N}$ (27) $\displaystyle-\gamma_{i}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{inter}-\gamma_{t}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{temporal}.$ $$

where $\gamma_{n}$, $\gamma_{i}$, and $\gamma_{t}$ denote the strengths of these guidance, respectively.
As shown in the experiment section, all three guidance terms are essential to jointly achieve spatial accuracy, temporal consistency, and physical plausibility for 4D garment reconstruction.

Finally, we map the predicted $\hat{\mathbf{x}}_{0}=[\mathbf{C},\mathbf{D}^{g}]$ to the UV space to obtain a partial positional map $\tilde{\mathbf{U}}$ and a binary mask $\tilde{\mathbf{M}}$, as shown in the bottom-right of Fig. [5](#S4.F5). These maps are then used in the next section to recover the full garment deformations.

**TABLE I: Quantitative comparison with SOTA methods. “$\dagger$” denotes models that use refinement. Bold: best; Underline: second best.**
| Method | Skirt | Trousers | Tshirt | Open Shirt |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CD $\downarrow$ | NC $\uparrow$ | IoU $\uparrow$ | CD $\downarrow$ | NC $\uparrow$ | IoU $\uparrow$ | CD $\downarrow$ | NC $\uparrow$ | IoU $\uparrow$ | CD $\downarrow$ | NC $\uparrow$ | IoU $\uparrow$ |  |
| SMPLicit [10] | 5.80 | 0.34 | 53.72 | 1.46 | -0.12 | 74.53 | 2.77 | 0.09 | 58.64 | 2.40 | 0.03 | 54.81 |
| ISP [41] | 4.18 | 0.78 | 68.10 | 1.53 | 0.78 | 75.39 | 1.36 | 0.82 | 68.70 | 1.69 | 0.79 | 71.31 |
| GaRec$\dagger$ [40] | 2.67 | 0.88 | 96.33 | 1.67 | 0.85 | 92.55 | 1.19 | 0.83 | 87.15 | 1.21 | 0.79 | 91.40 |
| D^3-Human$\dagger$ [7] | 3.64 | 0.78 | 76.42 | 1.80 | 0.79 | 82.49 | 1.79 | 0.79 | 78.51 | 1.71 | 0.76 | 56.60 |
| DMap-Static (Ours) | 2.26 | 0.89 | 88.80 | 1.36 | 0.84 | 86.23 | 1.06 | 0.83 | 80.20 | 1.29 | 0.83 | 78.28 |
| DMap-Static$\dagger$ (Ours) | 2.17 | 0.91 | 94.80 | 0.85 | 0.88 | 94.64 | 0.81 | 0.88 | 93.34 | 0.97 | 0.83 | 89.85 |
| DMap-Dynamic (Ours) | 1.62 | 0.93 | 90.93 | 1.21 | 0.83 | 88.72 | 0.92 | 0.86 | 91.95 | 1.11 | 0.85 | 81.75 |
| DMap-Dynamic$\dagger$ (Ours) | 1.54 | 0.92 | 96.73 | 0.83 | 0.87 | 95.41 | 0.76 | 0.92 | 96.46 | 0.93 | 0.86 | 93.47 |

Figure: Figure 7: Qualitative comparison on an in-the-wild image. The top and bottom rows show the front and the back of the reconstructions produced by our method DMap, BCNet [28], SMPLicit [10], ISP [41], GaRec [40] and ECON [69], respectively.
Refer to caption: 2602.24043v1/x6.png

### V-C Projection-Based Constraint for Deformation Modeling

To model the completed garment deformation while maximally preserving the established observation $\tilde{\mathbf{U}}$, we introduce a projection-based constraint, DDPMp, inspired by [67]. This constraint enforces strong alignment between the generation process and the incomplete positional map $\tilde{\mathbf{U}}$. We apply DDPMp on the deformation prior $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$ of DISP and enforce temporal smoothness as

$$ $\displaystyle\mathbf{x}_{t-1}=$ $\displaystyle\text{DDPM}_{p}\!\left(\mathbf{x}_{t},\,\tilde{\mathbf{U}},\,\tilde{\mathbf{M}},\,\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t)\right)$ (28) $\displaystyle-\gamma_{v}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{vel}^{d}-\gamma_{a}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{acc}^{d}\;,$ $$

where $\gamma_{v}$ and $\gamma_{a}$ are the guidance scales. $\text{DDPM}_{p}$ projects the estimate into observed and unobserved components using range-null space decomposition. To denoise only on the unobserved part, we write

$$ $\displaystyle\text{DDPM}_{p}(\mathbf{x}_{t},\tilde{\mathbf{U}},\tilde{\mathbf{M}},\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t))=\;\frac{\sqrt{\bar{\alpha}_{t-1}}\beta_{t}}{1-\bar{\alpha}_{t}}\hat{\mathbf{x}}_{0\mid t}$ (29) $\displaystyle+\frac{\sqrt{\alpha_{t}}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_{t}}\mathbf{x}_{t}+\sigma_{t}\boldsymbol{\epsilon},\quad\boldsymbol{\epsilon}\sim\mathcal{N}(0,\mathbf{I}).$ $$

$$ $\hat{\mathbf{x}}_{0\mid t}=\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{U}}+\left(\mathbf{I}-\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{M}}\right)\mathbf{x}_{0\mid t}\;,$ (30) $$

where $\tilde{\mathbf{M}}^{\dagger}$ is the pseudo-inverse of the observation mask $\tilde{\mathbf{M}}$, $\mathbf{x}_{0\mid t}$ is the estimated $\mathbf{x}_{0}$ at timestep $t$ following DDPM [25]. Intuitively, $\tilde{\mathbf{M}}^{\dagger}$ projects $\mathbf{x}_{0\mid t}$ to the range-space of $\tilde{\mathbf{M}}$, satisfying the observation $\tilde{\mathbf{M}}\hat{\mathbf{x}}_{0\mid t}\equiv\tilde{\mathbf{U}}$, while the $(\mathbf{I}-\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{M}})$ operator projects $\mathbf{x}_{0\mid t}$ to the null-space of $\tilde{\mathbf{M}}$, which does not affect the observation but determines whether $\hat{\mathbf{x}}_{0\mid t}$ follows the prior distribution. This projection-based constraint is crucial here, as it generates positional maps $\hat{\mathbf{U}}=\mathbf{x}_{0}$ by maximally aligning the extracted observations $\tilde{\mathbf{U}}$ and satisfies the learned diffusion priors that maintain harmony between the observed and unobserved regions,
resulting in a plausible and complete reconstruction. More detailed derivations and explanations can be found in Sec. B of the Supplementary Material.

As in Sec. [V-B1](#S5.SS2.SSS1), we introduce velocity and acceleration guidance $\mathcal{G}^{d}_{vel}$ and $\mathcal{G}^{d}_{acc}$ to ensure temporal smoothness. They are computed from the current frame and previous frames $\hat{\mathbf{U}}_{f-1}$/$\hat{\mathbf{U}}_{f-2}$ as

$$ $\mathcal{G}^{d}_{vel}=||\mathbf{x}_{0\mid t}-{\hat{\mathbf{U}}}_{f-1}||_{2}^{2},$ (31) $$

$$ $\mathcal{G}^{d}_{acc}=||\mathbf{x}_{0\mid t}-2\hat{\mathbf{U}}_{f-1}+\hat{\mathbf{U}}_{f-2}||_{2}^{2}\;.$ (32) $$

The update of Eq. [28](#S5.E28) yields a sequence of complete positional maps, from which we can recover the mesh sequence $\mathbf{G}$ as illustrated in the bottom-left of Fig. [5](#S4.F5).

### V-D Refinement

To bridge the domain gap between synthetic training data and real-world videos, we adopt the refinement strategy in Sec. [IV-C3](#S4.SS3.SSS3) and extend it to account for video dynamics by introducing a temporal optimization term

$$ $\mathcal{L}_{{temporal}}=\mathcal{L}_{{vel}}+\mathcal{L}_{{acc}},$ (33) $$

which penalizes irregular velocity and acceleration, and enforces temporal consistency in the reconstructed sequence.

Figure: Figure 8: Qualitative comparison on a video sequence. Our method (second row) achieves finer geometric detail, better temporal consistency, and avoids the garment–body collisions frequently observed in REC-MV [55] (bottom row), where the reconstructed garment often penetrates the body surface.
Refer to caption: 2602.24043v1/x7.png

## VI Experiments

### VI-A Dataset and Evaluation Metrics

#### VI-A 1 Dataset

Due to the intricate structure of garments, collecting real 3D data with complete geometry for them is extremely difficult. Instead, we use physics-based simulation to generate garment with realistic deformations in its interaction with the underlying body.
CLOTH3D [2] is a synthetic dataset, with 3D garments draped on T-posed SMPL bodies [47]. For each clothing category, including shirt, open shirt, skirt and trousers, we randomly select 33 samples. Each pair of garments and body models is simulated with the motion data sourced from the dance category of the AMASS dataset [50]. The motion sequences are generated by using Blender [4] and Marvelous Designer [14]. Additional pins are manually set for open shirt, skirt, and trousers to avoid sliding during the simulation. For each body sample in the sequence, we randomly rotate it and render its front and back body part segmentation and depth images. For the corresponding garment sample, we rotate it with the same angle and render its front and back normal and depth images. Its front and back UV coordinate images are generated using the UV parameterization of ISP. For each garment category, we randomly select 30 pairs of garment and body for training and use the rest pairs for the evaluation. For DMap-Static, we subsample the training data by selecting one frame every five frames, whereas for DMap-Dynamic, we do not subsample and instead use consecutive frames for training.

#### VI-A 2 Evaluation Metrics

To evaluate the quality of garment reconstruction, we use the Chamfer Distance (CD) and the Normal Consistency (NC) between the ground truth and the recovered garment mesh, and the Intersection over Union (IoU) between the ground truth mask and the rendered mask of reconstructed garment mesh. The quantitative comparison is conducted on the synthetic test set, while qualitative evaluation is performed on in-the-wild images and videos that include loose garments, diverse motions, and occlusions.

Figure: Figure 9: Reconstruction for in-the-wild images. Our method can recover realistic 3D models for diverse garments in different shapes and deformations.
Refer to caption: 2602.24043v1/x8.png

Figure: Figure 10: Qualitative results on real-world videos. For each sample, we show the input, the front- and back-view reconstructions, and the 2D renderings.
Refer to caption: 2602.24043v1/x9.png

### VI-B Reconstruction Results

#### VI-B 1 Quantitative Comparison

In Table [I](#S5.T1), we report the quantitative comparison between our methods and state-of-the-art approaches.
SMPLicit [10], ISP [41], and GaRec [40] are image-based methods, whereas D^3-Human [7] is a video-based approach.
We first evaluate the inferred versions (DMap-Static, DMap-Dynamic), obtained directly via diffusion inference. DMap-Static already outperforms existing methods, while DMap-Dynamic achieves even higher accuracy by considering the spatio-temporal consistency jointly. We also present results for the refined versions (DMap-Static^†, DMap-Dynamic^†), where an additional refinement stage (Sec. [IV-C](#S4.SS3) and [V-D](#S5.SS4)) is applied to further enhance the reconstruction quality.
Notably, for loose-fitting skirts, DMap-Dynamic^† delivers the best performance by a substantial margin, demonstrating the effectiveness of our spatio-temporal diffusion and refinement in capturing large garment deformations. Please see Sec. A of the Supplementary Material and the supplementary video for dynamic comparisons on video sequences.

#### VI-B 2 Qualitative Comparison

Fig. [7](#S5.F7) shows a qualitative comparison on an in-the-wild image for the static reconstruction. Methods such as BCNet[28], SMPLicit, and ISP rely on body-driven skinning and therefore keep garments tightly attached to the body. ECON [69] and GaRec can recover garments that stand away from the body surface, but ECON merges the body and garment into a single watertight mesh, and GaRec  [40] often yields flat surfaces with limited fold details. In contrast, our method uses back-normal estimation together with DISP priors to reconstruct garment meshes with realistic wrinkles and high-fidelity details on both the front and back.
Fig. [8](#S5.F8) presents a qualitative comparison on a video sequence with the state-of-the-art video-based method REC-MV [55]. Our method reconstructs garments with richer geometric detail and better temporal consistency, while also avoiding garment-body collisions that are clearly visible in the results of REC-MV.

Figure: Figure 11: Ablation of fitting strategy. (a) The input image and its normal estimations for the front and back. (b) Our full reconstruction. (c) Reconstruction without post-refinement. (d) Reconstruction refined by optimizing only the neural displacement field. (e) Reconstruction refined by optimizing only the vertex positions. (f) Reconstruction using only the front normal.
Refer to caption: 2602.24043v1/x10.png

#### VI-B 3 Reconstructions on real-world images and videos

Fig. [9](#S6.F9) provides more results of our method applied to in-the-wild images, demonstrating its ability to produce realistic 3D meshes with fine details for both tight-fitting and loose-fitting garments. Fig. [10](#S6.F10) provides additional qualitative results on real-world video sequences with various garment types, including T-shirts, skirts, trousers, and jackets. These results demonstrate that our method can handle a wide range of garments and human poses while ensuring physically plausible interactions between them. Notably, the rendered images closely align with the input frames, and the geometric details are consistent with the images, indicating that our method effectively extracts and preserves visual observations. Please refer to the supplementary video for dynamic results.

**TABLE II: Inference time. “$\dagger$” denotes models that use refinement.**
| Category | Method | Per-Frame Time |
| --- | --- | --- |
| Image-based | SMPLicit [10] | 8 min |
| ISP [41] | 19 min |  |
| GaRec$\dagger$ [40] | 1 min |  |
| DMap-Static | 7 min |  |
| DMap-Static$\dagger$ | 24 min |  |
| Video-based | REC-MV$\dagger$ [55] | 12 min |
| D^3-Human$\dagger$ [7] | 13 min |  |
| DMap-Dynamic | 3 min |  |
| DMap-Dynamic$\dagger$ | 7 min |  |

#### VI-B 4 Inference Time

Table [II](#S6.T2) presents the runtime comparison. DMap-Dynamic achieves lower computational cost than most competing methods. This efficiency primarily stems from our sequential formulation, which enables effective parallelization across frames. In contrast, prior approaches, including video-based methods such as REC-MV [55] and D^3-Human [7], process each frame independently and therefore do not exploit sequence-level parallelism.
Although GaRec [40] attains faster per-frame inference, its reconstruction accuracy is substantially lower, as evidenced in Table [I](#S5.T1), where our method surpasses all baselines.
DMap-Dynamic requires $3$ minutes for base inference and, optionally, $7$ minutes when the refinement stage is applied. This design offers a practical balance between efficiency and accuracy: the inference-only mode provides rapid reconstruction, whereas the refinement stage can be enabled when maximal geometric fidelity is desired.

**TABLE III: Ablation study. w/o $N_{B}$ means optimization without using the back normal estimation $N_{B}$. $f_{\phi}$ denotes optimization using the neural displacement field, and $V$ denotes optimization conducted directly on vertex positions.**
|  | CD $\downarrow$ | IoU $\uparrow$ | NC $\uparrow$ |
| --- | --- | --- | --- |
| w/o $N_{B}$ | 1.67 | 90.04 | 0.78 |
| w/o $f_{\phi}$, w/o $V$ | 1.45 | 92.67 | 0.81 |
| w/o $f_{\phi}$, w/ $V$ | 1.32 | 93.34 | 0.82 |
| w/ $f_{\phi}$, w/o $V$ | 1.27 | 94.71 | 0.82 |
| Ours | 1.21 | 95.32 | 0.83 |

**TABLE IV: Ablation study of the proposed spatial guidance in DMap-dynamic on the CLOTH3D dataset.**
| Method | Skirt | Trousers | Tshirt | Open Shirt |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| CD$\downarrow$ | NC$\uparrow$ | IoU$\uparrow$ | CD$\downarrow$ | NC$\uparrow$ | IoU$\uparrow$ | CD$\downarrow$ | NC$\uparrow$ | IoU$\uparrow$ | CD$\downarrow$ | NC$\uparrow$ | IoU$\uparrow$ |  |
| inference w/o back view | 2.67 | 0.90 | 83.62 | 1.90 | 0.81 | 79.09 | 1.61 | 0.81 | 78.31 | 2.00 | 0.73 | 70.46 |
| inference w/o $\mathcal{G}_{\mathrm{D2N}}$ | 1.97 | 0.92 | 90.67 | 1.53 | 0.82 | 86.53 | 1.24 | 0.84 | 85.73 | 1.56 | 0.82 | 72.55 |
| inference w/o $\mathcal{G}_{\mathrm{inter}}$ | 1.71 | 0.92 | 90.53 | 1.30 | 0.82 | 86.62 | 1.07 | 0.83 | 88.23 | 1.24 | 0.83 | 77.63 |
| inference | 1.62 | 0.93 | 90.93 | 1.21 | 0.83 | 88.72 | 0.92 | 0.86 | 91.95 | 1.11 | 0.85 | 81.75 |
| inference + refinement | 1.54 | 0.92 | 96.73 | 0.83 | 0.87 | 95.41 | 0.76 | 0.92 | 96.46 | 0.93 | 0.86 | 93.47 |

### VI-C Ablation Study

#### VI-C 1 Effectiveness of Fitting Strategy

Fig. [11](#S6.F11) presents the ablation study of our fitting method of DMap-Static introduced in Sec. [IV-C](#S4.SS3). As shown in Fig. [11](#S6.F11) (c), the initial reconstruction, without the post-refinement step described in Section [IV-C3](#S4.SS3.SSS3), fails to fully align with the input image shown in Fig. [11](#S6.F11) (a). Incorporating a neural displacement field to optimize the initial mesh improves reconstruction accuracy, as seen in Fig. [11](#S6.F11) (d). Further refinement by directly optimizing vertex positions enhances the wrinkle details, as illustrated in Fig. [11](#S6.F11) (b). However, applying post-refinement without first optimizing the neural displacement field (Fig. [11](#S6.F11) (e)) struggles to recover an accurate shape, as each vertex is optimized independently, leading to suboptimal results. Finally, Fig. [11](#S6.F11) (f) shows the outcome when only the front normal estimation is used throughout the fitting process described in Section [IV-C](#S4.SS3). The lack of constraints for the back surface results in unrealistic deformations on the back. The corresponding quantitative results are provided in Table [III](#S6.T3).

Figure: Figure 12: Ablation on spatial guidance. (a) 2D observations, including input image, front normal, and back normal. (b) Reconstruction after refinement. (c) Reconstruction after inference. (d) Inferred result without using generated back-view normals. (e) Inferred result without depth-to-normal guidance $\mathcal{G}_{D2N}$. (f) Inferred result without interpenetration-aware guidance $\mathcal{G}_{inter}$.
Refer to caption: 2602.24043v1/x11.png

Figure: Figure 13: Effectiveness of Temporal Components. We measure the acceleration of corresponding vertices between consecutive frames to quantify the temporal consistency of the garment motions.
Refer to caption: 2602.24043v1/x12.png

#### VI-C 2 Effectiveness of Spatial Guidance

Table [IV](#S6.T4) and Fig. [12](#S6.F12) show the ablation study of the spatial guidance terms of DMap-Dynamic introduced in Sec. [V-B2](#S5.SS2.SSS2). As shown in Fig. [12](#S6.F12) (d), without back-view guidance, the diffusion model tends to produce overly smooth garments that lack realistic draping. We hypothesize that, in the absence of both observations and guidance, the diffusion model is only able to generate an averaged garment mesh where all draping has been smoothed out. The depth-to-normal guidance $\mathcal{G}_{D2N}$ encourages the depth to capture normal details. Fig. [12](#S6.F12) (e) shows that removing this guidance leads to a loss of fine details and misalignment with the normal observations.
Finally, without the interpenetration-aware guidance $\mathcal{G}_{inter}$, the estimated garment collides with the body, as illustrated in Fig. [12](#S6.F12) (f).

#### VI-C 3 Effectiveness of Temporal Components

We evaluate temporal consistency by measuring vertex acceleration between consecutive frames.
Fig. [14](#S6.F14) shows the acceleration curves on a synthetic skirt sequence and illustrates the effect of different temporal components: the temporal diffusion module, across-subsequence guidance ($\mathcal{G}_{across}$), within-subsequence guidance ($\mathcal{G}_{within}$), and the temporal loss used during refinement. The black curve denotes ground-truth acceleration.
Without the temporal module (orange), DMap-Dynamic reduces to an image-based method and exhibits large, frequent accelerations, indicating strong motion jitter. Incorporating the temporal module (purple) substantially reduces jitter, though inconsistencies remain at subsequence boundaries.
Across-subsequence guidance (pink) mitigates these boundary artifacts, and combining $\mathcal{G}_{across}$ with $\mathcal{G}_{within}$ (red) further improves temporal smoothness during inference.
Finally, the comparison between the deep and light green curves highlights the importance of the refinement-stage temporal loss $\mathcal{L}_{temporal}$ for achieving the highest level of temporal coherence.

#### VI-C 4 Effectiveness of Projection-Based Inpainting

Fig. [14](#S6.F14) compares the inpainted reconstructions from the incomplete positional map $\tilde{\mathbf{U}}$ using our projection-based constraints and a naive constraints that directly minimizing $||\tilde{\mathbf{M}}(\mathbf{x}_{0\mid t}-\tilde{\mathbf{U}})||_{2}^{2}$.
For better visualization, we map the positional map to point clouds.
It can be observed that the reconstruction with projection constraint aligns more closely with the sparse input, thereby better preserving the accuracy and temporal consistency established in the previous stages. In contrast, the naive constraint provides weaker control over the generation process, which
introduces discrepancies and leads to spatial and temporal inconsistencies.

Figure: Figure 15: Retargeting. Left: The input image and our reconstructions. Right: The reconstructed garments are transferred to body with different poses and shapes.
Refer to caption: 2602.24043v1/x14.png

### VI-D Downstream Applications

##### Retargeting

Since our method produces separate models for the garment and the underlying body, we can easily repose it on the new body. In Fig. [16](#S6.F16), we show the retargeting results for the reconstructed open shirt and trousers by transferring them onto bodies with different poses and shapes. Accurate reconstruction of garments results in realistic retargeting.

##### Texture Editing

Since we reconstruct both the 3D model and the corresponding 2D panels for garment, we can easily realize texture editing. As shown in Fig. [16](#S6.F16), by painting patterns or drawing specific figures onto the recovered panels, the mesh will show the texture on the corresponding position.

## VII Conclusion

We have proposed a unified spatio-temporal framework for high-fidelity 3D and 4D garment generation from monocular visual input. When using single images as input, it delivers accurate static reconstructions even for loose-fitting clothing without requiring predefined templates. To this end, it uses UV-based sewing patterns to bridge the gap between 2D image observations and the 3D geometry of loose-fitting garments and relies on diffusion schemes to infer the parts of the garments that cannot be seen.
When processing videos, our method extends the single-image diffusion scheme to a spatio-temporal formulation with test-time guidance that enforces long-range temporal consistency. For the more challenging spatio-temporal modeling of unobserved regions, we introduce analytic projection-based constraints that faithfully preserve visible garment geometry while enforcing spatial and temporal consistency in occluded areas.
Extensive experiments on benchmarks and real-world imagery demonstrate that our method consistently outperforms existing approaches on both static and dynamic garments, achieving high fidelity and strong temporal consistency.

In future work, we will further improve efficiency by revisiting our spatio-temporal diffusion formulation that, like most diffusion-based video methods, generates results by denoising variables initialized from a Gaussian distribution. Temporal consistency is then promoted by minimizing guidance terms during generation. As an alternative, we will explore a sequential formulation based on diffusion or flow matching, where the estimate from the previous snippet initializes the current one. The current snippet would then be obtained by deforming the preceding shape estimate while conditioning on the current 2D observations and physical constraints. Such warm-started sequential generation may reduce repeated denoising and yield more natural temporal coherence across snippets.

## Acknowledgments

This work was supported in part by the Swiss National Science Foundation grant and by the Metaverse Center Grant from the MBZUAI Research Office.

## References

- [1]
G. Bae and A. Davison (2024)
Rethinking Inductive Biases for Surface Normal Estimation.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§IV-A](#S4.SS1.p1.6).
- [2]
H. Bertiche, M. Madadi, and S. Escalera (2020)
CLOTH3D: Clothed 3D Humans.
In European Conference on Computer Vision,
pp. 344–359.
Cited by: [§III-A](#S3.SS1.SSS0.Px2.p1.1),
[§VI-A1](#S6.SS1.SSS1.p1.1).
- [3]
B. L. Bhatnagar, G. Tiwari, C. Theobalt, and G. Pons-Moll (2019)
Multi-Garment Net: Learning to Dress 3D People from Images.
In International Conference on Computer Vision,
pp. 5420–5430.
Cited by: [§I](#S1.p1.1),
[§I](#S1.p2.1),
[§II-A](#S2.SS1.p1.1).
- [4]
(2018)
Blender.
Note: https://www.blender.org/
Cited by: [§VI-A1](#S6.SS1.SSS1.p1.1).
- [5]
F. Bogo, A. Kanazawa, C. Lassner, P. Gehler, J. Romero, and M. J. Black (2016)
Keep It SMPL: Automatic Estimation of 3D Human Pose and Shape from a Single Image.
In European Conference on Computer Vision,
pp. 561–578.
Cited by: [§I](#S1.p1.1).
- [6]
A. Casado-Elvira, M. Trinidad, and D. Casas (2022)
PERGAMO: Personalized 3D Garments from Monocular Video.
In Computer Graphics Forum,
pp. 293–304.
Cited by: [§I](#S1.p2.1),
[§II-A](#S2.SS1.p1.1).
- [7]
H. Chen, B. Peng, Y. Tao, and J. Zhang (2025)
D3-Human: Dynamic Disentangled Digital Human from Monocular Video.
In arXiv Preprint,
Cited by: [§I](#S1.p4.1),
[§II-B](#S2.SS2.p1.1),
[TABLE I](#S5.T1.17.15.2),
[§VI-B1](#S6.SS2.SSS1.p1.4),
[§VI-B4](#S6.SS2.SSS4.p1.3),
[TABLE II](#S6.T2.7.5.2).
- [8]
H. Chung, J. Kim, M. T. Mccann, M. L. Klasky, and J. C. Ye (2022)
Diffusion Posterior Sampling for General Noisy Inverse Problems.
In International Conference on Learning Representations,
Cited by: [§IV-C2](#S4.SS3.SSS2.p1.5),
[§V-B1](#S5.SS2.SSS1.p1.9).
- [9]
H. Chung, B. Sim, D. Ryu, and J. C. Ye (2022)
Improving Diffusion Models for Inverse Problems Using Manifold Constraints.
In Advances in Neural Information Processing Systems,
pp. 25683–25696.
Cited by: [§IV-C2](#S4.SS3.SSS2.p1.5).
- [10]
E. Corona, A. Pumarola, G. Alenya, G. Pons-Moll, and F. Moreno-Noguer (2021)
Smplicit: Topology-Aware Generative Model for Clothed People.
In Conference on Computer Vision and Pattern Recognition,
pp. 11875–11885.
Cited by: [§I](#S1.p1.1),
[§II-A](#S2.SS1.p1.1),
[Figure 7](#S5.F7),
[TABLE I](#S5.T1.19.19.1.1),
[§VI-B1](#S6.SS2.SSS1.p1.4),
[TABLE II](#S6.T2.8.8.2.2).
- [11]
R. Danerek, E. Dibra, C. öztireli, R. Ziegler, and M. Gross (2017)
Deepgarment: 3D Garment Shape Estimation from a Single Image.
In Computer Graphics Forum,
pp. 269–280.
Cited by: [§I](#S1.p1.1),
[§I](#S1.p2.1),
[§II-A](#S2.SS1.p1.1).
- [12]
S. Dasgupta, S. Naik, P. Savalia, S. K. Ingle, and A. Sharma (2025)
NGD: neural gradient based deformation for monocular garment reconstruction.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§I](#S1.p4.1),
[§I](#S1.p6.1),
[§II-B](#S2.SS2.p1.1).
- [13]
L. DeLuigi, R. Li, B. Guillard, M. Salzmann, and P. Fua (2023)
Drapenet: Generating Garments and Draping Them with Self-Supervision.
In Conference on Computer Vision and Pattern Recognition,
pp. 1451–1460.
Cited by: [§II-A](#S2.SS1.p1.1).
- [14]
M. Designer (2018)
Note: [https://www.marvelousdesigner.com](https://www.marvelousdesigner.com)
Cited by: [§VI-A1](#S6.SS1.SSS1.p1.1).
- [15]
P. Esser, J. Chiu, P. Atighehchian, J. Granskog, and A. Germanidis (2023)
Structure and Content-Guided Video Synthesis with Diffusion Models.
In International Conference on Computer Vision,
Cited by: [§II-C](#S2.SS3.p1.1).
- [16]
M. Gadelha, R. Wang, and S. Maji (2021)
Deep Manifold Prior.
In International Conference on Computer Vision,
Cited by: [§IV-C3](#S4.SS3.SSS3.p2.7).
- [17]
S. Goel, G. Pavlakos, J. Rajasegaran, A. Kanazawa, and J. Malik (2023)
Humans in 4D: Reconstructing and Tracking Humans with Transformers.
In International Conference on Computer Vision,
pp. 14783–14794.
Cited by: [§IV-A](#S4.SS1.p1.6).
- [18]
B. Guillard, F. Stella, and P. Fua (2022)
Meshudf: Fast and Differentiable Meshing of Unsigned Distance Field Networks.
In European Conference on Computer Vision,
pp. 576–592.
Cited by: [§II-A](#S2.SS1.p1.1).
- [19]
C. Guo, T. Jiang, X. Chen, J. Song, and O. Hilliges (2023)
Vid2Avatar: 3D Avatar Reconstruction from Videos in the Wild via Self-Supervised Scene Decomposition.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§II-B](#S2.SS2.p1.1).
- [20]
C. Guo, T. Jiang, M. Kaufmann, C. Zheng, J. Valentin, J. Song, and O. Hilliges (2024)
ReLoo: Reconstructing Humans Dressed in Loose Garments from Monocular Video in the Wild.
In European Conference on Computer Vision,
Cited by: [§I](#S1.p4.1),
[§I](#S1.p6.1),
[§II-B](#S2.SS2.p1.1).
- [21]
C. Guo, J. Li, Y. Kant, Y. Sheikh, S. Saito, and C. Cao (2025)
Vid2Avatar-Pro: Authentic Avatar from Videos in the Wild via Universal Prior.
In arXiv Preprint,
Cited by: [§II-B](#S2.SS2.p1.1).
- [22]
J. Guo, F. Prada, D. Xiang, J. Romero, C. Wu, H. S. Park, T. Shiratori, and S. Saito (2024)
Diffusion Shape Prior for Wrinkle-Accurate Cloth Registration.
In International Conference on 3D Vision,
pp. 790–799.
Cited by: [§II-C](#S2.SS3.p1.1),
[§III-B](#S3.SS2.p1.6).
- [23]
Y. Guo, C. Yang, A. Rao, Z. Liang, Y. Wang, M. Agrawala, D. Lin, and B. Dai (2023)
Animatediff: Animate Your Personalized Text-To-Image Diffusion Models Without Specific Tuning.
In arXiv Preprint,
Cited by: [§II-C](#S2.SS3.p1.1).
- [24]
J. Ho, A. Jain, and P. Abbeel (2020)
Denoising Diffusion Probabilistic Models.
Advances in Neural Information Processing Systems 33, pp. 6840–6851.
Cited by: [§III-B](#S3.SS2.p2.5).
- [25]
J. Ho, A. Jain, and P. Abbeel (2020)
Denoising Diffusion Probabilistic Models.
In Advances in Neural Information Processing Systems,
Cited by: [§II-C](#S2.SS3.p1.1),
[§V-C](#S5.SS3.p1.23),
[§VIII-B](#S8.SS2.p2.14),
[§VIII-B](#S8.SS2.p2.4),
[§VIII-B](#S8.SS2.p3.16),
[§VIII-D](#S8.SS4.SSS0.Px2.p1.8).
- [26]
J. Ho and T. Salimans (2022)
Classifier-Free Diffusion Guidance.
In arXiv Preprint,
Cited by: [§II-C](#S2.SS3.p1.1).
- [27]
B. Jiang, Y. Hong, H. Bao, and J. Zhang (2022)
SelfRecon: Self Reconstruction Your Digital Avatar from Monocular Video.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§II-B](#S2.SS2.p1.1).
- [28]
B. Jiang, J. Zhang, Y. Hong, J. Luo, L. Liu, and H. Bao (2020)
Bcnet: Learning Body and Cloth Shape from a Single Image.
In European Conference on Computer Vision,
pp. 18–35.
Cited by: [§I](#S1.p1.1),
[§I](#S1.p2.1),
[§II-A](#S2.SS1.p1.1),
[Figure 7](#S5.F7),
[§VI-B2](#S6.SS2.SSS2.p1.1),
[§VIII-A](#S8.SS1.SSS0.Px2.p1.1).
- [29]
X. Jiang, Z. Li, R. Missel, M. S. Zaman, B. Zenger, W. W. Good, R. S. MacLeod, J. L. Sapp, and L. Wang (2022)
Few-Shot Generation of Personalized Neural Surrogates for Cardiac Simulation via Bayesian Meta-Learning.
In Conference on Medical Image Computing and Computer Assisted Intervention,
pp. 46–56.
Cited by: [§II-B](#S2.SS2.p1.1).
- [30]
Z. Jiang, C. Guo, M. Kaufmann, T. Jiang, J. Valentin, O. Hilliges, and J. Song (2024)
Multiply: Reconstruction of Multiple People from Monocular Video in the Wild.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§II-B](#S2.SS2.p1.1).
- [31]
A. Kanazawa, M. J. Black, D. W. Jacobs, and J. Malik (2018)
End-To-End Recovery of Human Shape and Pose.
In Conference on Computer Vision and Pattern Recognition,
pp. 7122–7131.
Cited by: [§I](#S1.p1.1).
- [32]
R. Khirodkar, T. Bagautdinov, J. Martinez, S. Zhaoen, A. James, P. Selednik, S. Anderson, and S. Saito (2024)
Sapiens: Foundation for Human Vision Models.
In European Conference on Computer Vision,
Cited by: [§VIII-D](#S8.SS4.SSS0.Px2.p1.8).
- [33]
R. Khirodkar, T. Bagautdinov, J. Martinez, S. Zhaoen, A. James, P. Selednik, S. Anderson, and S. Saito (2025)
Sapiens: Foundation for Human Vision Models.
In European Conference on Computer Vision,
Cited by: [§IV-A](#S4.SS1.p1.6).
- [34]
D. P. Kingma and J. Ba (2015)
Adam: A Method for Stochastic Optimisation.
In International Conference on Learning Representations,
Cited by: [§VIII-D](#S8.SS4.SSS0.Px1.p3.3).
- [35]
A. Kirillov, E. Mintun, N. Ravi, H. Mao, C. Rolland, L. Gustafson, T. Xiao, S. Whitehead, A. C. Berg, W.-Y. Lo, P. Dollár, and R. Girshick (2023)
Segment Anything.
In arXiv Preprint,
Cited by: [§IV-A](#S4.SS1.p1.6).
- [36]
C. Lassner, J. Romero, M. Kiefel, F. Bogo, M.J. Black, and P.V. Gehler (2017)
Unite the People: Closing the Loop Between 3D and 2D Human Representations.
In Conference on Computer Vision and Pattern Recognition,
pp. 6050–6059.
Cited by: [§I](#S1.p1.1).
- [37]
P. Li, Y. Xu, Y. Wei, and Y. Yang (2020)
Self-Correction for Human Parsing.
IEEE Transactions on Pattern Analysis and Machine Intelligence 44 (6), pp. 3260–3271.
Cited by: [§IV-A](#S4.SS1.p1.6).
- [38]
R. Li, C. Cao, C. Dumery, Y. You, H. Li, and P. Fua (2025)
Single View Garment Reconstruction Using Diffusion Mapping via Pattern Coordinates.
In SIGGRAPH Conference Papers ’25,
Cited by: [§I](#S1.p6.1),
[§II-A](#S2.SS1.p1.1).
- [39]
R. Li, C. Dumery, Z. Dang, and P. Fua (2024)
Reconstruction of Manipulated Garment with Guided Deformation Prior.
In Advances in Neural Information Processing Systems,
Cited by: [§II-C](#S2.SS3.p1.1),
[§III-B](#S3.SS2.p1.6),
[Figure A](#S8.F1),
[§VIII-A](#S8.SS1.SSS0.Px1.p1.1).
- [40]
R. Li, C. Dumery, B. Guillard, and P. Fua (2024)
Garment Recovery with Shape and Deformation Priors.
In Conference on Computer Vision and Pattern Recognition,
pp. 1586–1595.
Cited by: [§I](#S1.p3.1),
[§III-A](#S3.SS1.SSS0.Px2.p1.1),
[Figure 7](#S5.F7),
[TABLE I](#S5.T1.15.13.1),
[§VI-B1](#S6.SS2.SSS1.p1.4),
[§VI-B2](#S6.SS2.SSS2.p1.1),
[§VI-B4](#S6.SS2.SSS4.p1.3),
[TABLE II](#S6.T2.3.1.1),
[§VIII-A](#S8.SS1.SSS0.Px2.p1.1),
[§VIII-D](#S8.SS4.SSS0.Px1.p1.5).
- [41]
R. Li, B. Guillard, and P. Fua (2023)
ISP: Multi-Layered Garment Draping with Implicit Sewing Patterns.
In Advances in Neural Information Processing Systems,
pp. 40294–40319.
Cited by: [§I](#S1.p3.1),
[§II-A](#S2.SS1.p1.1),
[§III-A](#S3.SS1.SSS0.Px1.p1.6),
[§III-A](#S3.SS1.SSS0.Px2.p1.1),
[§III](#S3.p1.1),
[Figure 7](#S5.F7),
[TABLE I](#S5.T1.19.20.2.1),
[§VI-B1](#S6.SS2.SSS1.p1.4),
[TABLE II](#S6.T2.8.9.3.1),
[§VIII-D](#S8.SS4.SSS0.Px1.p1.5).
- [42]
R. Li, B. Guillard, E. Remelli, and P. Fua (2022)
DIG: Draping Implicit Garment over the Human Body.
In Asian Conference on Computer Vision,
pp. 2780–2795.
Cited by: [§II-A](#S2.SS1.p1.1).
- [43]
W. Li, M. Liu, H. Liu, B. Ren, X. Li, Y. You, and N. Sebe (2024)
HYRE: hybrid regressor for 3d human pose and shape estimation.
IEEE Transactions on Image Processing.
Cited by: [§I](#S1.p1.1).
- [44]
X. Li, J. Zhang, Y. Lai, J. Yang, and K. Li (2023)
High-Quality Animatable Dynamic Garment Reconstruction from Monocular Videos.
IEEE Transactions on Circuits and Systems for Video Technology.
Cited by: [§II-B](#S2.SS2.p1.1).
- [45]
L. Liu, L. Zhang, Y. Xu, C. Gotsman, and S. J. Gortler (2008)
A Local/global Approach to Mesh Parameterization.
In Proceedings of the Symposium on Geometry Processing,
pp. 1495–1504.
Cited by: [§III-A](#S3.SS1.SSS0.Px2.p1.1).
- [46]
X. Liu, J. Li, and G. Lu (2023)
Modeling Realistic Clothing from a Single Image Under Normal Guide.
IEEE Transactions on Visualization and Computer Graphics 30 (7), pp. 3995–4007.
Cited by: [§I](#S1.p2.1),
[§II-A](#S2.SS1.p1.1).
- [47]
M. Loper and M.J. Black (2014)
Opendr: An Approximate Differentiable Renderer.
In European Conference on Computer Vision,
pp. 154–169.
Cited by: [§VI-A1](#S6.SS1.SSS1.p1.1).
- [48]
M. Loper, N. Mahmood, J. Romero, G. Pons-Moll, and M.J. Black (2015)
SMPL: A Skinned Multi-Person Linear Model.
ACM SIGGRAPH Asia 34 (6), pp. 1–16.
Cited by: [§I](#S1.p2.1),
[§IV-A](#S4.SS1.p1.6).
- [49]
Z. Luo, D. Chen, Y. Zhang, Y. Huang, L. Wang, Y. Shen, D. Zhao, J. Zhou, and T. Tan (2023)
Videofusion: Decomposed Diffusion Models for High-Quality Video Generation.
In arXiv Preprint,
Cited by: [§II-C](#S2.SS3.p1.1).
- [50]
N. Mahmood, N. Ghorbani, N. F. Troje, G. Pons-Moll, and M. J. Black (2019)
AMASS: Archive of Motion Capture as Surface Shapes.
In International Conference on Computer Vision,
pp. 5442–5451.
Cited by: [§VI-A1](#S6.SS1.SSS1.p1.1).
- [51]
G. Moon, H. Nam, T. Shiratori, and K.M. Lee (2022)
3D Clothed Human Reconstruction in the Wild.
In European Conference on Computer Vision,
pp. 184–200.
Cited by: [§I](#S1.p1.1),
[§II-A](#S2.SS1.p1.1).
- [52]
R. Narain, A. Samii, and J.F. O’brien (2012)
Adaptive Anisotropic Remeshing for Cloth Simulation.
ACM Transactions on Graphics 31 (6), pp. 1–10.
Cited by: [§IV-C3](#S4.SS3.SSS3.p1.16).
- [53]
P. Paudel, A. Khanal, D. P. Paudel, J. Tandukar, and A. Chhatkuli (2024)
Ihuman: Instant Animatable Digital Humans from Monocular Videos.
In European Conference on Computer Vision,
Cited by: [§II-B](#S2.SS2.p1.1).
- [54]
N. Pietroni, C. Dumery, R. Falque, M. Liu, T. Vidal-Calleja, and O. Sorkine-Hornung (2022)
Computational Pattern Making from 3D Garment Models.
ACM Transactions on Graphics 41 (4), pp. 1–14.
Cited by: [§III-A](#S3.SS1.SSS0.Px2.p1.1).
- [55]
L. Qiu, G. Chen, J. Zhou, M. Xu, J. Wang, and X. Han (2023)
REC-MV: Reconstructing 3D Dynamic Cloth from Monocular Videos.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§I](#S1.p4.1),
[§II-B](#S2.SS2.p1.1),
[Figure 8](#S5.F8),
[§VI-B2](#S6.SS2.SSS2.p1.1),
[§VI-B4](#S6.SS2.SSS4.p1.3),
[TABLE II](#S6.T2.5.3.1).
- [56]
N. Rahaman, A. Baratin, D. Arpit, F. Draxler, M. Lin, F. Hamprecht, Y. Bengio, and A. Courville (2019)
On the Spectral Bias of Neural Networks.
In International Conference on Machine Learning,
Cited by: [§IV-C3](#S4.SS3.SSS3.p2.9).
- [57]
N. Ravi, V. Gabeur, Y. Hu, R. Hu, C. Ryali, T. Ma, H. Khedr, R. Rädle, C. Rolland, L. Gustafson, and L. A. Others (2024)
Sam 2: Segment Snything in Images and Videos.
In arXiv Preprint,
Cited by: [§IV-A](#S4.SS1.p1.6).
- [58]
O. Ronneberger, P. Fischer, and T. Brox (2015)
U-Net: Convolutional Networks for Biomedical Image Segmentation.
In Conference on Medical Image Computing and Computer Assisted Intervention,
pp. 234–241.
Cited by: [§VIII-D](#S8.SS4.SSS0.Px1.p2.32).
- [59]
I. Santesteban, M.A. Otaduy, and D. Casas (2022)
SNUG: Self-Supervised Neural Dynamic Garments.
In Conference on Computer Vision and Pattern Recognition,
pp. 8140–8150.
Cited by: [§IV-C3](#S4.SS3.SSS3.p1.16),
[§VIII-D](#S8.SS4.SSS0.Px3.p1.1).
- [60]
S. Shin, J. Kim, E. Halilaj, and M. J. Black (2024)
Wham: Reconstructing World-Grounded Humans with Accurate 3D Motion.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§VIII-D](#S8.SS4.SSS0.Px2.p1.8).
- [61]
S. Shin, J. Kim, E. Halilaj, and M. J. Black (2024)
WHAM: reconstructing world-grounded humans with accurate 3D motion.
In CVPR,
pp. 2070–2080.
Cited by: [§IV-A](#S4.SS1.p1.6).
- [62]
U. Singer, A. Polyak, T. Hayes, X. Yin, J. An, S. Zhang, Q. Hu, H. Yang, O. Ashual, O. Gafni, et al. (2022)
Make-A-Video: Text-To-Video Generation Without Text-Video Data.
In arXiv Preprint,
Cited by: [§II-C](#S2.SS3.p1.1).
- [63]
J. Song, C. Meng, and S. Ermon (2020)
Denoising Diffusion Implicit Models.
arXiv Preprint.
Cited by: [§II-C](#S2.SS3.p1.1).
- [64]
A. Stathopoulos, L. Han, and D. Metaxas (2024)
Score-Guided Diffusion for 3D Human Recovery.
In Conference on Computer Vision and Pattern Recognition,
pp. 906–915.
Cited by: [§IV-A](#S4.SS1.p1.6).
- [65]
D. Ulyanov, A. Vedaldi, and V. Lempitsky (2018)
Deep Image Prior.
In Conference on Computer Vision and Pattern Recognition,
pp. 9446–9454.
Cited by: [§IV-C3](#S4.SS3.SSS3.p2.7).
- [66]
S. Wang, K. Schwarz, A. Geiger, and S. Tang (2022)
Arah: Animatable Volume Rendering of Articulated Human Sdfs.
In European Conference on Computer Vision,
Cited by: [§II-B](#S2.SS2.p1.1).
- [67]
Y. Wang, J. Yu, and J. Zhang (2022)
Zero-Shot Image Restoration Using Denoising Diffusion Null-Space Model.
In arXiv Preprint,
Cited by: [§V-C](#S5.SS3.p1.5),
[§VIII-B](#S8.SS2.p2.4).
- [68]
C. Weng, B. Curless, P. P. Srinivasan, J. T. Barron, and I. Kemelmacher-Shlizerman (2022)
Humannerf: Free-Viewpoint Rendering of Moving People from Monocular Video.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [§II-B](#S2.SS2.p1.1).
- [69]
Y. Xiu, J. Yang, X. Cao, D. Tzionas, and M. J. Black (2023)
ECON: Explicit Clothed Humans Optimized via Normal Integration.
In Conference on Computer Vision and Pattern Recognition,
Cited by: [Figure 7](#S5.F7),
[§VI-B2](#S6.SS2.SSS2.p1.1).
- [70]
J. Xu, Y. Guo, and Y. Peng (2024)
FinePOSE: fine-grained prompt-driven 3d human pose estimation via diffusion models.
In Conference on Computer Vision and Pattern Recognition,
pp. 561–570.
Cited by: [§I](#S1.p1.1).

## VIII Supplementary Material

This supplementary material contains the following parts:

(A) Additional Qualitative Results.

(B) Range-Null Space Decomposition and DDPMp.

(C) Recovering Rest Geometry with Accumulated Masks.

(D) Implementation Details.

(E) Supplementary Video ‘supplementary_video.mp4’.

### VIII-A Additional Qualitative Results

##### Static Reconstruction

In Fig. [A](#S8.F1), we show additional comparison with GaRec [39]. DMap-Static can capture high-fidelity surface details and complex deformations, while GaRec [39] produces results overly smooth and lack realistic wrinkles. In Fig. [B](#S8.F2), we show more reconstruction results produced by DMap-Static for in-the-wild images. As illustrated, DMap-Static accurately reconstructs garments, capturing intricate deformations and fine surface details.

Figure: Figure A: Comparison between DMap-Static and GaRec [39].
Refer to caption: 2602.24043v1/x16.png

Figure: Figure B: More reconstruction results for in-the-wild images. DMap-Static can handle both the tight-fitting and the loose-fitting garments and recover high-fidelity 3D meshes for them.
Refer to caption: 2602.24043v1/x17.png

Figure: Figure C: Additional qualitative comparison for in-the-wild videos. DMap-Dynamic achieves both geometric accuracy and temporal consistency.
Refer to caption: 2602.24043v1/x18.png

##### Dynamic Reconstruction

Fig. [C](#S8.F3) presents additional qualitative comparisons on real-world video sequences with previous methods, including BCNet [28], GaRec [40], and DMap-Static. These examples are particularly challenging due to fast motions, loose-fitting garments, and occlusions. BCNet is limited to frontal views and struggles to reconstruct accurate garment types or body poses. GaRec often fails under back or side views, lacking spatiotemporal consistency. Although DMap-Static achieves high per-frame accuracy, it also fails to model spatio-temporal consistency, producing jittery results as illustrated in the supplementary video.
Besides, it tends to produce visual artifacts during garment motion. In contrast, DMap-Dynamic achieves the best overall performance, combining per-frame accuracy, physical plausibility and temporal consistency.
Please refer to the supplementary video for dynamic comparisons.

##### Visualization of Error

Fig. [D](#S8.F4) and [E](#S8.F5) visualize the spatial error distribution on reconstructed garment meshes, measured using one-directional Chamfer Distance. Our method achieves lower error across the entire surface, whereas competing methods exhibit significant errors, particularly in regions farther from the underlying body.

Figure: Figure D: Visualization of error distribution on the reconstructed jacket mesh. The bright colors indicate large errors.
Refer to caption: 2602.24043v1/x19.png

Figure: Figure E: Visualization of error distribution on the reconstructed skirt mesh. The bright colors indicate large errors.
Refer to caption: 2602.24043v1/x20.png

### VIII-B Range-Null Space Decomposition and DDPM p

Given a linear operator $\mathbf{A}$, any sample $\mathbf{x}$ can be split into range- and null-space parts of $\mathbf{A}$:

$$ $\mathbf{x}\equiv\underbrace{\mathbf{A}^{\dagger}\mathbf{A}\mathbf{x}}_{\text{range-space part}}+\underbrace{(\mathbf{I}-\mathbf{A}^{\dagger}\mathbf{A})\mathbf{x}}_{\text{null-space part}},$ (34) $$

where $\mathbf{A}^{\dagger}$ is the pseudo-inverse of $\mathbf{A}$. The matrices $\mathbf{A}$ and $\mathbf{A}^{\dagger}$ exhibit useful projection properties. Specifically, $\mathbf{A}^{\dagger}\mathbf{A}$ projects $\mathbf{x}$ onto the range space of $\mathbf{A}$, since $\mathbf{A}\mathbf{A}^{\dagger}\mathbf{A}\mathbf{x}\equiv\mathbf{A}\mathbf{x}$. Conversely, $(\mathbf{I}-\mathbf{A}^{\dagger}\mathbf{A})$ projects $\mathbf{x}$ onto the null space of $\mathbf{A}$, as $\mathbf{A}(\mathbf{I}-\mathbf{A}^{\dagger}\mathbf{A})\mathbf{x}\equiv\mathbf{0}$.

Inspired by [67], which uses range-null space decomposition in the DDPM [25] denoising process to solve the inverse problems in 2D images, we introduce such projection-based constraint in the positional representation and temporal guidance to recover the garment deformations by completing positional map from the partial observation $\tilde{\mathbf{U}}$ and its corresponding mask $\tilde{\mathbf{M}}$. We aim for the final output $\mathbf{x}_{0}$ to both satisfy the partial observation constraint $\tilde{\mathbf{M}}\mathbf{x}_{0}\equiv\tilde{\mathbf{U}}$ and remain plausible with temporal consistency.
The denoising step is written as:

$$ $\mathbf{x}_{t-1}=\text{DDPM}_{p}(\mathbf{x}_{t},\tilde{\mathbf{U}},\tilde{\mathbf{M}},\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t))-\gamma_{v}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{vel}^{d}-\gamma_{a}\nabla_{\mathbf{x}_{t}}\mathcal{G}_{acc}^{d}\;,$ (35) $$

where $\gamma_{v}$ and $\gamma_{a}$ are the guidance scales. $\text{DDPM}_{p}$ projects the estimate into observed and unobserved components using range-null space decomposition. Specifically, during the denoising process, we denote $\mathbf{x}_{0|t}$ as the predicted clean sample at timestep $t$ computed from the noisy sample $\mathbf{x}_{t}$:

$$ $\mathbf{x}_{0\mid t}=\frac{1}{\sqrt{\bar{\alpha}_{t}}}\left(\mathbf{x}_{t}-\sqrt{1-\bar{\alpha}_{t}}\cdot\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t)\right),$ (36) $$

where $\bar{\alpha}_{t}$ is the scale factor. This equation retains equivalence with the original DDPM [25]. To constrain the final generation to satisfy the partial observation $\tilde{\mathbf{M}}\mathbf{x}_{0}\equiv\tilde{\mathbf{U}}$, we modify $\mathbf{x}_{0\mid t}$ by fixing the range-space as $\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{U}}$ and leaving the null-space unchanged:

$$ $\hat{\mathbf{x}}_{0\mid t}=\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{U}}+\left(\mathbf{I}-\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{M}}\right)\mathbf{x}_{0\mid t}\;,$ (37) $$

where $\tilde{\mathbf{M}}^{\dagger}$ is the pseudo-inverse of the observation mask $\tilde{\mathbf{M}}$. Intuitively, $\tilde{\mathbf{M}}^{\dagger}$ projects $\mathbf{x}_{0\mid t}$ to the range-space of $\tilde{\mathbf{M}}$, satisfying the observation $\tilde{\mathbf{M}}\hat{\mathbf{x}}_{0\mid t}\equiv\tilde{\mathbf{U}}$, while $(\mathbf{I}-\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{M}})$ projects $\mathbf{x}_{0\mid t}$ to the null-space of $\tilde{\mathbf{M}}$, which does not affect the observation but determines whether $\hat{\mathbf{x}}_{0\mid t}$ follows the prior distribution.

Then, DDPMp rewrites the sampling formulation from posterior distribution $p\left(\mathbf{x}_{t-1}\mid\mathbf{x}_{t},\mathbf{x}_{0}\right)$ of DDPM by replacing the $\mathbf{x}_{0}$ with it estimate $\hat{\mathbf{x}}_{0\mid t}$, which is written as:

$$ $\text{DDPM}_{p}(\mathbf{x}_{t},\tilde{\mathbf{U}},\tilde{\mathbf{M}},\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}(\mathbf{x}_{t},t))=\frac{\sqrt{\bar{\alpha}_{t-1}}\beta_{t}}{1-\bar{\alpha}_{t}}\hat{\mathbf{x}}_{0\mid t}+\frac{\sqrt{\alpha_{t}}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_{t}}\mathbf{x}_{t}+\sigma_{t}\boldsymbol{\epsilon},$ (38) $$

where $\boldsymbol{\epsilon}\sim\mathcal{N}(0,\mathbf{I})$, and $\alpha$, $\bar{\alpha}$, $\beta$ are scales factors predefined in DDPM [25]. The property of DDPMp is that the range-space component $\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{U}}$ remains fixed, while only the null-space component $(\mathbf{I}-\tilde{\mathbf{M}}^{\dagger}\tilde{\mathbf{M}})\mathbf{x}_{0\mid t}$ undergoes denoising to eliminate disharmony between the two components. By iteratively denoising using Eqs. [35](#S8.E35), [36](#S8.E36), [37](#S8.E37), [38](#S8.E38), we obtain a final result $\mathbf{x}_{0}$ that satisfies the partial observation $\tilde{\mathbf{M}}\mathbf{x}_{0}\equiv\tilde{\mathbf{U}}$ while remaining within the distribution $\mathbf{x}_{0}\sim q(\mathbf{x})$. This projection-based constraint is crucial here,
as it generates positional maps $\hat{\mathbf{U}}$ by maximally aligning the extracted observations $\tilde{\mathbf{U}}$ and satisfies the learned diffusion priors that maintain harmony between the observed and unobserved regions, resulting in a plausible and complete reconstruction.

### VIII-C Recovering Rest Geometry with Accumulated Partial Masks for Dynamic Reconstruction

Figure: Figure F: Recovering the rest garment using accumulated partial masks in DMap-Dynamic.
Refer to caption: 2602.24043v1/x21.png

For the dynamic reconstruction, instead of using $\tilde{\mathbf{M}}$ from a single image as introduced in Sec. [IV-C1](#S4.SS3.SSS1), we accumulate the partial masks across all frames to form a more complete representation $\tilde{\mathbf{M}}_{acc}={\bigvee}_{f=1}^{T}\tilde{\mathbf{M}}_{f}$ in DMap-Dynamic, since the observed masks from different frames offer globally complementary coverage.

We optimize the latent code of the garment prior so that the decoded panel patterns closely match $\tilde{\mathbf{M}}_{acc}$ (Eq. [11](#S4.E11)). The resulting optimal latent code is subsequently used to infer the complete panel mask $\hat{\mathbf{M}}$ and the corresponding rest garment mesh, as shown in Fig. [F](#S8.F6).

### VIII-D Implementation Details

##### Networks and Training

Following [40, 41], we implement the pattern parameterization network $\mathcal{I}_{\Theta}$ and the UV parameterization network $\mathcal{A}_{\Phi}$ of ISP as MLPs with the latent code $\mathbf{z}$ of size 128. $\mathcal{I}_{\Theta}$ and $\mathcal{A}_{\Phi}$ are trained jointly for 9000 iterations with a batch size of 50.

For the diffusion models $\boldsymbol{\epsilon}^{d}_{\boldsymbol{\theta}}$, $\boldsymbol{\epsilon}_{\theta}^{n}$ and $\boldsymbol{\epsilon}_{\theta}^{m}$ of DMap-Static, we use a U-Net architecture [58] with attention blocks in the middle. They all have 6 down-sampling blocks and 6 up-sampling blocks.
The numbers of out channels of each block are [128, 128, 256, 256, 512, 512].
The output of $\boldsymbol{\epsilon}_{\theta}^{d}$ is the concatenated front and back UV maps and panel maps with the dimensions of $128\times 256\times 4$, and the input is the noise images having the same size as the output.
The output of $\boldsymbol{\epsilon}_{\theta}^{n}$ is the concatenation of an estimated normal image for the garment back and a mask indicating foreground pixels with the size of $192\times 192\times 4$. The input of $\boldsymbol{\epsilon}_{\theta}^{n}$ is the concatenation of [$\mathbf{n}_{F}$, $\mathbf{s}_{F}$, $\mathbf{s}_{B}$, $\mathbf{d}_{F}^{b}$, $\mathbf{d}_{B}^{b}$] and the noise image, whose size is $192\times 192\times 11$.
The output of $\boldsymbol{\epsilon}_{\theta}^{m}$ is the estimation of UV coordinates ($\mathbf{c}_{F}$, $\mathbf{c}_{B}$) and depth ($\mathbf{d}_{F}^{g}$, $\mathbf{d}_{B}^{g}$) with the size of $192\times 192\times 8$, and the input is the concatenation of [$\mathbf{n}_{F}$, $\mathbf{n}_{B}$, $\mathbf{s}_{F}$, $\mathbf{s}_{B}$, $\mathbf{d}_{F}^{b}$, $\mathbf{d}_{B}^{b}$] and the noise image, having the size of $192\times 192\times 18$.
$\boldsymbol{\epsilon}_{\theta}^{d}$ is trained for 100 epochs on frames sampled every 5 frames, with a learning rate of 1e-4, a batch size of 64, and $T=1000$ steps. We apply random rotations to the mesh represented as UV maps for augmentation.
$\boldsymbol{\epsilon}_{\theta}^{n}$ and $\boldsymbol{\epsilon}_{\theta}^{m}$ are trained for 200 epochs on the subsampled set of frames, with a learning rate of 1e-4, a batch size of 162, and $T=1000$ steps.

The temporal module of DMap-Dynamic for $\boldsymbol{\epsilon}_{\theta}^{n}$ and $\boldsymbol{\epsilon}_{\theta}^{m}$ consists of two linear layers around a temporal attention block, which is trained for 5 epochs on consecutive frames without subsampling, using a sequence length of $T=10$. The learning rate is set to 1e-4, with 500 warm-up steps.
All the models are trained using the Adam optimizer [34] on NVIDIA A100 GPUs.

The neural displacement field $f_{\phi}$ introduced in Sec. [IV-C3](#S4.SS3.SSS3) is a 4-layer MLP with Softplus activations in-between. The output channels of each layer is [256, 256, 256, 3]. For each garment, we optimize a separate randomly-initialized MLP.

##### Inference

We follow the DDPM formulation [25] with 1000 denoising timesteps. During sampling of DMap-Dynamic, we introduce multiple guidance terms as detailed in Sec. [V](#S5), with weights set as $\lambda_{a}=20$, $\lambda_{w}=1$, $\gamma=50$, $\gamma_{n}=5$, $\gamma_{i}=50$, $\gamma_{t}=500$, $\gamma_{v}=0.5$, and $\gamma_{a}=0.5$. For temporal diffusion, we divide the video into overlapping subsequences of 10 frames with a 5-frame overlap between adjacent segments.
At inference time on real-world videos of clothed humans, we employ Sapiens [32] to estimate front-view garment normal maps and WHAM [60] to predict human body meshes. These meshes are then rendered into body segmentations and depths, which serve as conditioning inputs.

**TABLE A: The values of hyper-parameters.**
| Eq. [11](#S4.E11) | $\lambda_{area}=0.5,\lambda_{\mathbf{z}}=0.02$ |
| --- | --- |
| Eq. [12](#S4.E12) | $\rho=20$ |
| Eq. [14](#S4.E14) | $\lambda_{m}=1,\lambda_{d}=2,\lambda_{n}=2,\lambda_{u}=100,\lambda_{p}=1$ |

##### Fitting

For the hyper-parameters introduced in Sec. [IV-C](#S4.SS3), we summarize their values in Table [A](#S8.T1). For the regularization term $\mathcal{L}_{phys}$ introduced in Eq. [16](#S4.E16), we adopt the default material parameters from [59] which are general enough to model most casually-worn garments.