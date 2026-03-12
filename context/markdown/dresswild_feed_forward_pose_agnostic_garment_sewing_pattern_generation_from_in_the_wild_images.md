<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - 2.1. 3D Reconstruction
  - 2.2. Garment Modeling and Generation
  - 2.3. Sewing Pattern Generation
- 3. Method
  - 3.1. Overview
  - 3.2. Data Processing
    - Sewing Pattern Representation
    - VLM-Guided Data Curation
  - 3.3. Feature Extraction
    - Canonicalization and Segmentation.
    - 3D Reconstruction Features.
    - Pose Features.
  - 3.4. Model
    - Feature Fusion Module.
    - Parameter Decoding.
    - Training Objective.
- 4. Post Process
  - 4.1. Texture Generation
    - Texture Generation.
  - 4.2. Garment Simulation
- 5. Experiments
  - 5.1. Experiment Settings
    - Implementation Details
    - Dataset
    - Baselines
  - 5.2. Quantitative Comparison
  - 5.3. Qualitative Comparison
  - 5.4. Ablation Study
    - Feature Extraction.
    - Feature Fusion Encoder.
- 6. Application
  - Sewing Pattern Generation from In-the-wild Images
  - Multi-layer Garment Generation
- 7. Conclusion
- References

## Abstract

Abstract. Recent advances in garment pattern generation have shown promising progress. However, existing feed-forward methods struggle with diverse poses and viewpoints, while optimization-based approaches are computationally expensive and difficult to scale. This paper focuses on sewing pattern generation for garment modeling and fabrication applications that demand editable, separable, and simulation-ready garments. We propose DressWild , a novel feed-forward pipeline that reconstructs physics-consistent 2D sewing patterns and the corresponding 3D garments from a single in-the-wild image. Given an input image, our method leverages vision–language models (VLMs) to normalize pose variations at the image level, then extract pose-aware, 3D-informed garment features. These features are fused through a transformer-based encoder and subsequently used to predict sewing pattern parameters, which can be directly applied to physical simulation, texture synthesis, and multi-layer virtual try-on. Extensive experiments demonstrate that our approach robustly recovers diverse sewing patterns and the corresponding 3D garments from in-the-wild images without requiring multi-view inputs or iterative optimization, offering an efficient and scalable solution for realistic garment simulation and animation.

## 1. Introduction

Traditional 3D garment creation follows a multi-stage pipeline that progresses from concept design to 2D sewing pattern drafting and finally to 3D garment assembly through virtual sewing and garment simulation. While this workflow enables precise control and fabrication-ready outputs, it is inherently time-consuming and requires substantial domain expertise, making it inaccessible to non-experts. Recent advances in visual computing and artificial intelligence have significantly lowered the barrier to 3D garment creation by enabling direct generation from images [Bian et al., 2025; Li et al., 2025c], text prompts [Li et al., 2025a; Zhou et al., 2025], or point clouds [Korosteleva and Lee, 2022], enabling efficient and scalable downstream virtual try-on and fabrication-aware garment design [Wu et al., 2025]. However, most existing approaches focus solely on producing visually plausible 3D garment geometry, without recovering the underlying 2D sewing patterns. The absence of pattern-level representations limits editability, parametric control, and physical manufacturability, posing a fundamental challenge for downstream design iteration and real-world fabrication.

Existing approaches for sewing-pattern-based garment generation can be broadly categorized into data-driven feed-forward methods [Liu et al., 2023b, 2025; Korosteleva and Lee, 2022] and optimization-based pipelines [Li et al., 2025e, 2024b]. Data-driven methods leverage paired pattern–garment datasets to directly predict sewing patterns or 3D garments but their performance is tightly coupled to the training distribution, often restricting them to canonical body poses such as A- or T-poses and limiting their ability to generalize to diverse pose sewing pattern. Moreover, these approaches typically require multiple input images or controlled capture conditions, reducing their practicality in real-world scenarios. Optimization-based methods, on the other hand, explicitly enforce physical consistency between sewing patterns and 3D garments and can support multi-pose configurations; however, they rely on iterative simulation and gradient-based optimization, resulting in high computational cost and long runtimes. These limitations motivate the need for a more efficient framework that can recover sewing patterns and physically consistent 3D garments from an image.

To address these challenges, we introduce DressWild, a feed-forward sewing-pattern generation pipeline that reconstructs pose-agnostic 2D sewing patterns from a single in-the-wild image, while producing the corresponding pose-fitted 3D garment by leveraging VLM priors. Given an in-the-wild input image, we first leverage a VLM to generate an auxiliary standard T-pose 2D representation of the dressed person. We then perform garment segmentation on both the canonical T-pose image and the original input image, and extract reconstruction features from the segmented outputs to capture the garment’s structural and appearance cues while suppressing background clutter.
In parallel, we perform pose feature extraction on the input image to explicitly encode body articulation. These features are subsequently integrated through our feature fusion module with a hybrid attention strategy, which injects VLM priors while selectively attending to complementary structure and pose cues. We predict the 2D sewing pattern representation based on the fused features. To summarize, our contributions are:

- •
We propose a feed-forward garment reconstruction pipeline that predicts diverse 2D sewing patterns and the corresponding physically consistent 3D garments, accurately fitted to people in arbitrary poses from a single in-the-wild image.
- •
We introduce a feature fusion and hybrid attention design that effectively incorporates VLM priors, enabling robust sewing pattern recovery and garment reconstruction under challenging multi-pose configurations.
- •
We conduct extensive experiments demonstrating the effectiveness and versatility of our approach, successfully reconstructing diverse sewing patterns and multi-pose 3D garments from in-the-wild images.

Figure: Figure 2. Overview of our pipeline. Given a single in-the-wild image, our method reconstructs simulation-ready sewing patterns and a corresponding 3D garment.
Refer to caption: x2.png

## 2. Related Work

### 2.1. 3D Reconstruction

With the emergence of powerful generative models, diffusions, transformers, autoregressive models and GANs have been widely adopted to synthesize 3D shapes from sparse conditioning signals such as images [Voleti et al., 2024; Qian et al., 2023; Wang et al., 2025a; Xiang et al., 2025], videos [Cong et al., 2025], text prompts [Chen et al., 2023; Poole et al., 2022], and sketches [Li et al., 2025b]. Existing training strategies for 3D generative models generally fall into two categories: 3D supervision and 2D prior guidance. The former learns a conditional mapping from paired sparse inputs to 3D ground truth representations (e.g., meshes [Wang et al., 2018], voxels [Xie et al., 2019], point clouds [Wang et al., 2025a], or implicit fields [Yu et al., 2021]), but typically relies on large-scale aligned 2D–3D datasets. The latter leverages multi-view rendering and differentiable rendering objectives derived from pretrained 2D priors, yet often requires additional constraints to ensure multi-view consistency [Huang et al., 2025; Shi et al., 2023; Xu et al., 2023]. Despite strong performance, these data-driven approaches are still limited by data distributions, making reconstruction from in-the-wild data particularly challenging [Li et al., 2025e]. To enable more diverse 3D generation without requiring paired data, score distillation sampling (SDS) [Poole et al., 2022] and variational score distillation (VSD) [Wang et al., 2023] are introduced to optimize 3D representations under pretrained diffusion priors. However, such optimization-based methods require time-consuming iterative refinement for each instance, hindering scalability. Different from optimization-based approaches, we target an efficient and scalable feed-forward framework for garment generation from an in-the-wild image.

### 2.2. Garment Modeling and Generation

A wide range of representations has been explored for modeling garment geometry, spanning implicit 3D fields, such as unsigned distance fields (UDF) [Yu et al., 2024], signed distance fields (SDF) [Moon et al., 2022], neural radiance fields (NeRF) , and explicit 3D representations, including meshes, point clouds, and 3D Gaussian splats [Wang et al., 2025b; Li et al., 2025a; Sarafianos et al., 2025], as well as 2D sewing pattern representations [Liu et al., 2023b; He et al., 2024]. Among these, many existing 3D garment generation methods adopt template-based pipelines to produce garment geometry. Specifically, BCNet [Su et al., 2021], DeepGarment [Daněřek et al., 2017], GarNet [Gundogdu et al., 2019], and TailorNet [Patel et al., 2020] employ feed-forward neural networks trained on synthetic datasets to predict template-based garment deformations, represented as per-vertex offsets or low-dimensional deformation latents, which are subsequently applied to predefined garment meshes. However, these methods typically require category-specific training, with each model assuming a fixed garment topology and being trained separately for individual garment types. Alternatively, Liu et al. [2023c]; Li et al. [2025a]; Sarafianos et al. [2025]; Casado-Elvira et al. [2022] adopt optimization-based pipelines that refine garment mesh vertices at test time using guidance from multi-view images, 3D Gaussian Splatting, latent CLIP supervision, or normal maps. Nevertheless, these approaches still rely on predefined garment mesh templates for initialization, which inherently constrains the diversity of garment shapes they can represent. To enable more diverse garment generation, recent works explore generalized generative models that synthesize garments from images or text instead of training separate models for each garment category and using predefines mesh templates. SMPLicit [Corona et al., 2021] introduces an MLP-based generative model that predicts unsigned distance fields for implicit cloth modeling in the SMPL body space. ClothWild [Moon et al., 2022] extends this formulation to in-the-wild images, jointly reconstructing human shape and clothing geometry from a single view. Surf-D [Yu et al., 2024] further explores diffusion-based models to recover non-watertight garment geometry directly from images. These methods mainly target 3D garment geometry and might omit explicit 2D sewing patterns, thereby limiting their applicability to downstream simulation and animation. Recent works DressCode [He et al., 2024] and SewFormer [Liu et al., 2023b], explore transformer models to generate both 3D garments and corresponding 2D sewing patterns from text prompts and images, respectively. Our work also focuses on generating sewing patterns and their corresponding 3D garments.

### 2.3. Sewing Pattern Generation

The development of large-scale sewing pattern datasets, such as GarmentCodeData [Korosteleva et al., 2024] and SewFactory [Liu et al., 2023a], has enabled data-driven approaches to sewing pattern generation. Building on these datasets, prior work explores generative models to infer structured sewing pattern representations in both implicit pixel space and explicit vector space, capturing 2D pattern geometry as well as the associated 3D transformation parameters required for garment assembly and simulation. For instance, learning-based approaches explore diffusions, transformers, autoregressive models [He et al., 2024; Chen et al., 2022; Guo et al., 2025; Liu et al., 2025; Li et al., 2025d] to synthesize sewing patterns from images and text prompts. While these methods generate vectorized sewing pattern parameters, their discrete latent spaces can limit generalization to unseen garment styles. In contrast, Garment Image [Tatsukawa et al., 2025] represents sewing pattern geometry, topology, and placement in a rasterized form, yielding a more continuous latent space and improved generalization. Similarly, ISP [Li et al., 2023], GarmentRecovery [Li et al., 2024a], and DMap [Li et al., 2025c] adopt rasterized implicit sewing pattern representations to enable higher-quality garment reconstruction with improved efficiency. To further enable multimodal control for sewing pattern editing and garment generation from text, images, and sketches, recent methods such as AIpparel [Nakayama et al., 2025], Design2GarmentCode [Zhou et al., 2025], and ChatGarment [Bian et al., 2025] integrate large language models (LLMs), large multimodal models (LMMs), and vision–language models (VLMs), respectively, into the sewing pattern generation pipeline. However, these methods have difficulty generalizing to in-the-wild images, as reconstructed garments are often biased toward the training data distribution, resulting in misalignment with the input image and limiting the diversity and fidelity needed to capture real-world garment variations. To enable diverse garment generation, Dress-1-to-3 [Li et al., 2025e] and DiffAvatar [Li et al., 2024b] adopt differentiable frameworks that optimize sewing pattern geometric and physical parameters, which are predicted by transformers or specified by users, respectively. These approaches enable high-fidelity reconstruction of out-of-distribution garment shapes. However, their reliance on differentiable simulation can lead to failure cases under large human motions or extreme poses and incurs high computational costs. To complement prior work on pattern geometry, DeepIron [Kwon and Lee, 2023] predicts unwarped sewing-pattern textures from single images given fixed sewing pattern shapes. In contrast, our work aims to efficiently generate diverse and pose-agnostic, simulation-ready textured sewing patterns from in-the-wild images via fused multi-pose feature representations.

## 3. Method

### 3.1. Overview

As shown in Fig. [2](https://arxiv.org/html/2602.16502v1#S1.F2), we present DressWild, a novel framework for diverse and pose-agnostic sewing pattern generation that supports a wide range of garment categories, including T-shirts, jackets, pants, skirts, jumpsuits, dresses, and other apparel types. Given an input in-the-wild image $I$, which may exhibit arbitrary pose and viewpoint, DressWild first explore a VLM like Nanobanana Pro [Team et al., 2024] to transform the input image $I$ into a canonical representation by synthesizing a garment image in a fixed T-pose and front-facing view $I_{c}$. This normalization is achieved using a pre-trained VLM, which provides strong priors for disentangling pose and viewpoint variations from garment appearance. By leveraging VLM priors, our framework aligns the normalized image $I_{c}$ to the pose and view distribution of the training dataset while preserving garment-specific visual details. Given $I_{c}$ and $I$, we extract image appearance features $f_{i}$, and canonical-space normalized image features $f_{c}$ using Hunyuan3D [Team, 2025b]. In parallel, we employ SAM3D-Body [Yang et al., 2025], a state-of-the-art human reconstruction method, to extract pose-aware features $f_{p}$ and recover a 3D human mesh $M_{h}$ from the input image $I$. These embeddings are fused to form a joint feature

$$ (1) $f=\Phi(f_{p},f_{i},f_{c}),$ $$

where $\Phi(\cdot)$ denotes a feature fusion module. $f$ is further processed by a feed-forward garment modeling network to predict pose- and view-invariant sewing pattern parameters.
This design enables DressWild to leverage generative models trained on curated datasets while robustly generalizing to diverse, in-the-wild images without requiring multi-view or pose annotations. In the following sections, we elaborate on each component of the proposed pipeline.

### 3.2. Data Processing

#### Sewing Pattern Representation

We follow the sewing pattern templates from [Korosteleva and Lee, 2022] and [He et al., 2024], covering diverse garment categories. We represent a garment sewing pattern as a set of 2D panels together with their 3D placement and stitching topology. Specifically, a pattern consists of $N_{P}$ panels $\{P_{i}\}_{i=1}^{N_{P}}$, where each panel $P_{i}$ is defined as a closed planar polygon with an ordered set of vertices $\{\bm{v}_{i,k}\}_{k=1}^{N_{i}}\subset\mathbb{R}^{2}$. Consecutive vertices form edges $\{E_{i,j}\}_{j=1}^{N_{i}}$ that describe the panel boundary. Each edge is defined by two endpoints and an optional curvature term. Straight edges are represented as line segments, while curved edges are modeled using quadratic Bézier curves. A curved edge is parameterized by its start point $\bm{v}_{s}$, end point $\bm{v}_{e}$, and a single control point $\bm{c}\in\mathbb{R}^{2}$ specified in relative coordinates. To enable panel meshing and subsequent physical simulation, we discretize each curved sewing
pattern edge by sampling points along its quadratic Bézier representation. Specifically, a point
on a curved edge is computed as $\bm{B}(t)=(1-t)^{2}\bm{v}_{s}+2(1-t)t\bm{c}+t^{2}\bm{v}_{e},t\in[0,1]$, where uniformly sampling $t$ yields a set of boundary points that approximate the continuous
panel contour. The resulting discrete boundary enables robust panel triangulation for downstream cloth
simulation and stitching, while maintaining a compact and expressive representation of curved
garment contours. To place panels in 3D space, each panel $P_{i}$ is associated with a 6-DoF rigid transformation $(\bm{r}_{i},\bm{T}_{i})$, where $\bm{r}_{i}\in\mathbb{R}^{3}$ parameterizes the rotation and $\bm{T}_{i}\in\mathbb{R}^{3}$ denotes the translation, which defines its canonical 3D configuration. Stitching information is represented explicitly at the edge level using discrete stitching labels, where each label pairs two edges from different panels and encodes the sewing topology. Given an input image $I$, our network predicts the parameters of this structured representation, including panel vertex coordinates $\bm{v}_{i,k}$, edge curvature control points $\bm{c}_{i,j}$, panel-wise rigid transformations $(\bm{r}_{i},\bm{T}_{i})$, and stitching correspondences $\mathcal{S}$. By operating in this parametric sewing pattern space, the generated garments are pose- and view-invariant and directly compatible with physical simulation.

#### VLM-Guided Data Curation

We curate a sewing pattern dataset for multi-pose and multi-view garment images, aiming to support learning based garment modeling under diverse body configurations and camera viewpoints. Our dataset is built upon the dataset of [Korosteleva and Lee, 2021a], which covers more than 20,000 garment design variations derived from 19 base garment types. The original dataset provides T pose images from only two viewpoints. To extend the input images to cover more diverse poses and viewpoints, we leverage a vision language model (VLM) to synthesize multi-view and multi-pose images from front view T pose inputs. Specifically, we define three configurable parameter sets, $n_{v}$ viewpoints, $n_{p}$ body poses, and $n_{g}$ gestures, which are randomly combined to construct prompt style inputs for image generation. Although our data curation framework supports additional augmentation dimensions such as background scenes, lighting conditions, and clothing material or pattern descriptors, we do not adopt these settings in our final dataset construction. In practice, introducing such variations increases synthesis instability and can lead to inconsistencies between the generated images and underlying garment geometry. From a learning perspective, these factors primarily affect background appearance or materials rather than garment structure, resulting in limited performance gains. Therefore, we restrict augmentation to pose, view, and gesture level variations.

More specifically, the viewpoint configuration includes short keyword prompts such as ”low angle”, ”worm’s-eye view”, ”Dutch angle”, and ”eye-level”. The pose configuration is represented using concise descriptors including ”standing”, ”arching back”, ”jumping”, and ”walking”. The gesture configuration further augments motion diversity with prompt tokens such as ”arms crossed”, ”hands on hips”, and ”wiping sweat from the forehead”. This randomized combination of short, keyword-based prompts enables the generation of diverse and physically plausible variations while preserving garment structure and identity. In total, we define 10 distinct viewpoints, 21 body poses, and 21 gestures, whose randomized combinations are used as prompt tokens for multi-view and multi-pose image generation.

Figure: Figure 3. Data Curation and Augmentation. We use VLM to generate novel pose and novel view images with the consistent garment.
Refer to caption: x3.png

### 3.3. Feature Extraction

Given an in-the-wild input image $I$, our goal is to extract pose-aware and pose-invariant garment features for robust sewing pattern prediction. To this end, we construct three complementary feature streams capturing image appearance, canonical garment structure, and human pose.

#### Canonicalization and Segmentation.

We first leverage a VLM to normalize pose and viewpoint by synthesizing a canonical garment image $I_{c}$ in a fixed front-facing T-pose. This canonicalization step disentangles pose and view variations from garment appearance while preserving garment-specific visual details. We then apply HybridGL [Liu and Li, 2025] to segment both the original input image $I$ and the canonical image $I_{c}$, producing segmented garment images $I_{si}$ and $I_{sc}$, respectively.

#### 3D Reconstruction Features.

Based on the segmented images, we employ Hunyuan3D-2.0 [Team, 2025a] to extract 3D reconstruction features. Specifically, we utilize the intermediate latent representations produced after the Hunyuan3D-DiT sampling stage and before the VAE decoder, which capture rich 3D-aware garment geometry. From the segmented original image $I_{si}$, we extract multi-view multi-pose reconstruction features $f_{i}$, while from the canonical T-pose image $I_{sc}$, we obtain pose-normalized reconstruction features $f_{c}$. Both feature types are derived using the same reconstruction pipeline to ensure consistency.

#### Pose Features.

In parallel, we apply SAM3D-Body [Yang et al., 2025] to extract pose-aware embeddings $f_{p}$ by taking $I_{si}$ by the SAM3D-Body encoder. These features explicitly encode human pose information.

### 3.4. Model

Our model consists of two stages including feature fusion and parameter decoding. Multimodal features are fused into a pose and view invariant representation $f$, which is then decoded to predict structured sewing pattern parameters $\Theta$ including edge, stitching, rotation, and translation information.

#### Feature Fusion Module.

Since the three feature types originate from different encoders and exhibit heterogeneous dimensions and semantics, we first project each feature into a shared embedding space of dimension $d_{w}$ using separate linear transformations, $\tilde{f}_{i}=W_{i}f_{i},\tilde{f}_{c}=W_{c}f_{c},\tilde{f}_{p}=W_{p}f_{p}$
where $W_{i}$, $W_{c}$, and $W_{p}$ denote learnable projection matrices. This operation ensures that all features lie in the same feature space and can be jointly processed. We then concatenate the projected features along the sequence dimension, $\tilde{f}=[\tilde{f}_{i};\tilde{f}_{c};\tilde{f}_{p}],$
where the resulting sequence length equals $\lvert f_{i}\rvert+\lvert f_{c}\rvert+\lvert f_{p}\rvert$. The concatenated feature sequence is passed through a transformer encoder [Vaswani et al., 2017], which performs self-attention across all tokens to capture cross modal interactions among image appearance, canonical garment structure, and pose information. This mechanism enables the model to adaptively aggregate complementary cues from different feature sources, producing a fused representation $f=\Phi(\tilde{f})$, where $\Phi(\cdot)$ denotes the feature fusion module.

#### Parameter Decoding.

Given the fused feature representation $f$, we employ a decoder-based transformer [He et al., 2024] to autoregressively predict the sewing pattern parameters.

Each input token is formed by the sum of three embeddings: a positional embedding that identifies the panel associated with the token, a parameter type embedding that distinguishes edge geometry, rigid transformations, and stitching attributes, and a value embedding that encodes the quantized sewing pattern parameters.
Each output token represents a specific element of the sewing pattern parameterization, including panel vertex coordinates, edge curvature control points, 3D translation, rotation, and stitching correspondences. Through iterative decoding, the decoder attends to the fused feature representation and generates a sequence of latent tokens, which are subsequently decoded into the structured sewing pattern parameters.

$$ $\Theta=\{\{\bm{v}_{i,k}\},\{\bm{c}_{i,j}\},\{(\bm{r}_{i},\bm{T}_{i})\},\mathcal{S}\}.$ $$

By conditioning the decoder on pose and view invariant fused features, the predicted sewing patterns are robust to pose and viewpoint variations and are directly compatible with downstream physical simulation and animation.

#### Training Objective.

Our training objective consists of two components: a categorical cross entropy loss for discrete token prediction and a regression loss for continuous sewing pattern parameters. Our training objective is formulated as maximizing the joint conditional likelihood of discrete sewing pattern tokens and continuous geometric parameters. At each decoding step $t$, the decoder predicts a categorical distribution over discrete pattern parameter tokens, corresponding to the conditional likelihood

$$ $\mathcal{L}_{\text{CE}}=-\sum_{t=1}^{L}\log p(f_{t}\mid f_{<t},\,f;\theta),$ $$

where $f_{t}$ denotes the ground truth token at step $t$, $f_{<t}$ represents all previously generated tokens, and $f$ is the fused conditioning feature. This formulation is equivalent to the standard token-wise cross entropy loss used in autoregressive sequence modeling. In addition to discrete token prediction, we model the decoded continuous sewing pattern parameters using a Gaussian likelihood. After mapping the predicted tokens back to their continuous parameter values, we assume an isotropic Gaussian distribution over the continuous parameters, whose negative log likelihood reduces to a mean squared error loss:

$$ $\mathcal{L}_{\text{MSE}}=-\sum_{t=1}^{L}\log p(\bm{\theta}_{t}\mid f_{t},\,f;\theta)\propto\sum_{t=1}^{L}\lVert\hat{\bm{\theta}}_{t}-\bm{\theta}_{t}\rVert_{2}^{2},$ $$

where $\hat{\bm{\theta}}_{t}$ and $\bm{\theta}_{t}$ denote the predicted and ground truth continuous parameters at step $t$, respectively. This term encourages numerical accuracy of geometric quantities such as panel vertex coordinates and curvature control points. The final training objective is defined as a weighted sum of the two negative log likelihood terms:

$$ $\mathcal{L}=\mathcal{L}_{\text{CE}}+\lambda\mathcal{L}_{\text{MSE}},$ $$

where $\lambda$ balances discrete token likelihood maximization and continuous parameter regression.

## 4. Post Process

### 4.1. Texture Generation

#### Texture Generation.

The final step of DressWild focuses on generating high-fidelity textures for both sewing patterns, and their corresponding garment meshes, with the goal of supporting downstream garment fabrication. To this end, we introduce a dedicated texture generation module to reconstruct garment appearance. Unlike approaches that jointly generate geometry and texture, we explicitly decouple appearance synthesis from geometric reconstruction. Jointly predicting 2D sewing pattern textures and garment geometry is severely limited by the scarcity of paired supervision, making end-to-end learning difficult to scale. In addition, joint 3D texture and geometry generation from images is inherently ill-posed, as visual observations ambiguously entangle illumination, material appearance, and fine-scale geometric details, hindering reliable disentanglement of texture from shape.

Based on this design choice, we adopt a generative approach to first synthesize textures on the reconstructed 3D garment surface and subsequently transfer them to the sewing pattern domain. After obtaining sewing pattern parameters and generating a 3D garment mesh draped on the human body via physical simulation in Sec. [4.2](https://arxiv.org/html/2602.16502v1#S4.SS2), we extract multi-view consistent garment textures using Hunyuan3D-Paint [Team, 2025a], which goes through image delighting, multi-view consistent image generation, super-resolution and baking. The generated textures are then projected onto the UV parameterization induced by the sewing patterns, ensuring seam consistency and direct applicability to pattern-based fabrication workflows. This separation allows textures to be consistently aligned across seams, robust to pose-induced deformations, and directly applicable to physical garment production.

### 4.2. Garment Simulation

We adopt the SMPL-X model [Pavlakos et al., 2019] as the articulated human body representation and initialize garment placement around a T-posed body following Dress123 [Li et al., 2025e], using Magicman [He et al., 2025] to provide a coarse initial fit. As this rough fitting often introduces interpenetrations, we first apply Position-Based Dynamics (PBD) to project the garment mesh outside the human body while enforcing high stretching and bending stiffness to preserve the original shape. To ensure proper layering for multi-garment cases, stitched vertices are used to identify connected components, which are sorted vertically and sequentially fitted from bottom to top using the Codimensional Incremental Potential Contact (CIPC) simulator [Li et al., 2020], allowing upper garments to naturally overlay lower ones. The resulting configuration serves as an initial feasible state for CIPC-based dynamic simulation, after which the human body is interpolated from the T-pose to the reconstructed pose and simulated as a moving boundary condition. To prevent slippage of lower garments during pose interpolation, we locally shrink the rest shape of triangles near the waist to induce sufficient friction; the garment rest pose otherwise remains unchanged.

## 5. Experiments

### 5.1. Experiment Settings

#### Implementation Details

We employ NanoBanana Pro [Team et al., 2024] as the VLM. For image feature extraction, we utilize the pre-trained Hunyuan3D-2 [Team, 2025a] model. We employ the pre-trained HybridGL [Liu and Li, 2025] model for referring segmentation, and SAM3D-Body Dino-v3 [Yang et al., 2025] is used for pose feature extraction. Regarding the model architecture, the decoder consists of 24 layers with 8 attention heads. We use the Adam optimizer with an initial learning rate set to $10^{-4}$. The model is trained for 10 epochs on a single NVIDIA H100 GPU.

#### Dataset

We utilize [Korosteleva and Lee, 2021b] and our generated data to train and evaluate the models. The sewing patterns are categorized into 12 classes, which contain 25031 data points. We split the dataset into a training set, a validation set, and a testing set in a ratio of 7:2:1.

#### Baselines

We conduct both qualitative and quantitative comparisons with state-of-the-art sewing pattern generation methods, NeuralTailor and SewFormer, which generate sewing patterns from point clouds and single images, respectively.

### 5.2. Quantitative Comparison

We conduct quantitative comparisons with the baseline methods on the same test set across multiple evaluation metrics, including panel accuracy, edge accuracy, shape reconstruction error, Filtered shape (F-shape) error, and Chamfer Distance (CD). NeuralTailor relies on garment point cloud inputs, while SewFormer and our method predict sewing patterns directly from single in-the-wild images. For fair comparison, we use Hunyuan3D to generate the mesh from the input image and sample point clouds as inputs to NeuralTailor. As shown in Table 1, NeuralTailor performs poorly on in-the-wild images, achieving only 25.99% panel accuracy and 29.05% edge accuracy, which reflects its limited robustness to pose and viewpoint variations. SewFormer improves structural prediction to some extent, reaching 28.81% panel accuracy and 34.56% edge accuracy, but still struggles to recover correct panel topology and geometry under arbitrary poses. In contrast, our method substantially outperforms both baselines, achieving 94.35% panel accuracy and 85.41% edge accuracy. Moreover, our approach yields significantly lower geometric errors, reducing the $\ell_{2}$ shape error from 23.65 for NeuralTailor and 22.94 for SewFormer to 6.22, and achieving the lowest Chamfer Distance of 0.01899. Sewformer performs slightly better on F-shape error, for this metric ignores the wrong panels. These results demonstrate that our method produces both structurally correct and geometrically accurate sewing patterns from challenging in-the-wild images with diverse poses and viewpoints.

### 5.3. Qualitative Comparison

Figure: Figure 4. Qualitative Comparison. Our method produces high-quality 2D sewing patterns and corresponding 3D garments. Here we qualitatively compare our results with baseline approaches SewFormer [Liu et al., 2023b] and NeuralTailor [Korosteleva and Lee, 2022].
Refer to caption: x4.png

Fig. [4](https://arxiv.org/html/2602.16502v1#S5.F4) presents qualitative comparisons with baseline methods on the same test set described in Sec. [5.1](https://arxiv.org/html/2602.16502v1#S5.SS1), focusing on sewing pattern prediction from in-the-wild images with arbitrary poses and viewpoints. NeuralTailor, which relies on point cloud inputs, struggles to recover complete and structurally consistent pattern pieces, especially for garments with complex topology such as dresses and jackets. SewFormer shows reasonable performance under limited viewpoint variation, but its predictions often degrade for non-frontal or rotated poses, resulting in fragmented or misaligned panels. In contrast, our method consistently reconstructs coherent and well-proportioned sewing patterns across diverse in-the-wild conditions. By internally leveraging canonicalized representations together with original-view information, our approach better preserves garment topology, symmetry, and semantic part correspondence. These results demonstrate the robustness of our method to pose and viewpoint variations and its ability to recover simulation-ready sewing patterns from challenging in-the-wild images.

**Table 1. Quantitative Comparison. Our model outperforms the other methods on most of the metrics.**
| Method | Panel Acc. $\uparrow$ | Edge Acc. $\uparrow$ | Shape $\mathcal{L}^{2}$ $\downarrow$ | F-Shape $\mathcal{L}^{2}$ $\downarrow$ | CD $\downarrow$ |
| --- | --- | --- | --- | --- | --- |
| NeuralTailor | 25.99% | 29.05% | 23.65 | 7.30 | 0.02837 |
| Sewformer | 28.81% | 34.56% | 22.94 | 4.53 | 0.02797 |
| DressWild | 94.35% | 85.41% | 6.22 | 5.07 | 0.01899 |

Table. [1](https://arxiv.org/html/2602.16502v1#S5.T1) presents our quantitative results. Panel Acc. and Edge Acc. denote the prediction accuracy for the number of panels and edges, respectively, while Shape $\mathcal{L}^{2}$ Error represents the $\mathcal{L}^{2}$ distance of the parameters. Our method outperforms Sewformer across most of the metrics.

### 5.4. Ablation Study

We conduct an ablation study of the key components of our framework including additional visual features (canonical-space reconstruction features, pose features), and the feature fusion module, using the same garment images as in Sec. [5.1](https://arxiv.org/html/2602.16502v1#S5.SS1). This study assesses how each proposed component contributes to the final sewing pattern reconstruction quality.

Figure: Figure 5. Results from in-the-wild image with multi-layer garments. Given an in-the-wild image as input, our DressWild method can generate multi-layer pose-agnostic and simulation-ready sewing patterns.
Refer to caption: x5.png

**Table 2. Ablation of canonical-space reconstruction features. The performance of sewing pattern reconstruction is significantly improved with the inclusion of this feature.**
| Method | Panel Acc. $\uparrow$ | Edge Acc. $\uparrow$ | Shape $\mathcal{L}^{2}$ $\downarrow$ |
| --- | --- | --- | --- |
| w/o front features | 91.18% | 78.58% | 12.11 |
| w/ front features | 94.35% | 85.41% | 6.22 |

#### Feature Extraction.

We first evaluate the effect of different feature choices in our framework. As shown in Tab. [2](https://arxiv.org/html/2602.16502v1#S5.T2), incorporating canonical-space, front-view T-pose reconstruction features $f_{c}$ consistently improves both panel and edge accuracy, while reducing the shape error, indicating that front features provide critical geometric cues for recovering more accurate panel boundaries and overall sewing pattern shapes. Regarding pose features $f_{p}$ (Fig. [6](https://arxiv.org/html/2602.16502v1#S5.F6)), predictions that incorporate pose information remain much closer to the ground truth in both panel shape and overall layout, producing more plausible boundaries. By providing explicit pose information, the model is able to disentangle pose-induced garment deformation from the underlying, pose-invariant pattern geometry. In contrast, removing pose features causes the network to misinterpret draped appearances under non-neutral poses as intrinsic pattern geometry, leading to distorted or warped panels and incorrect proportions.

Figure: Figure 6. Ablation of pose features. The model identifies the garment better with the assistance of the pose feature when the human pose is challenging.

Figure: Figure 7. Ablation of feature fusion. With feature fusion design, the results are more consistent with GT.

#### Feature Fusion Encoder.

Since the features $f_{i}$, $f_{c}$, and $f_{p}$ originate from different encoders, we project them into a shared embedding space and fuse them with a transformer encoder using self-attention, producing a unified representation $f$. As shown in Fig. [7](https://arxiv.org/html/2602.16502v1#S5.F7), models equipped with the feature fusion module produce sewing pattern predictions that closely match the ground truth in both panel shape and overall layout. By projecting $f_{i}$, $f_{c}$, and $f_{p}$ into the shared embedding space, the model effectively captures cross-feature information. In contrast, removing the feature fusion module significantly degrades reconstruction quality, leading to incorrect proportions and distorted silhouettes, such as deformed collar shapes and inconsistent panel geometry. This degradation indicates that without explicit self-attention–based fusion, the model struggles to coherently aggregate complementary cues from different feature sources.

## 6. Application

#### Sewing Pattern Generation from In-the-wild Images

As illustrated in Fig. [5](https://arxiv.org/html/2602.16502v1#S5.F5), our method is capable of directly processing in-the-wild images exhibiting diverse poses, viewpoints. Given an in-the-wild input image, we apply DressWild to robustly generate pose-agnostic and simulation-ready sewing patterns.

#### Multi-layer Garment Generation

For garments composed of multiple layers, we follow a similar strategy by leveraging the VLM to decompose the input into individual garment layer image. Specifically, given an in-the-wild image containing layered apparel (e.g., a jacket worn over a shirt), we utilize the VLM to identify and extract each single-layer garment while preserving their relative ordering and semantic consistency. Each extracted garment layer image is then processed independently using DressWild to predict its corresponding pose-agnostic and simulation-ready sewing pattern as indicated in Fig. [5](https://arxiv.org/html/2602.16502v1#S5.F5). This design allows our framework to naturally extend from single-layer to multi-layer garment scenarios without requiring explicit layer annotations or modifications to the core model.

## 7. Conclusion

In this paper, we present DressWild, a novel feed-forward framework for reconstructing pose-agnostic, simulation-ready 2D sewing patterns and corresponding 3D garments from a single in-the-wild image. By leveraging VLMs to generate canonical garment representations and integrating pose-aware, canonical-space, and multi-view features via a hybrid mechanism, our approach effectively disentangles garment geometry from pose and viewpoint variations. Extensive experiments demonstrate that DressWild outperforms existing state-of-the-art methods in both quantitative metrics and qualitative comparison, particularly on challenging in-the-wild data with diverse poses and viewpoints. Furthermore, our pipeline supports multi-layer garment generation and produces physically plausible outputs compatible with downstream simulation and texture synthesis workflows, offering a scalable and efficient solution for realistic garment modeling and animation applications.

## References

- S. Bian, C. Xu, Y. Xiu, A. Grigorev, Z. Liu, C. Lu, M. J. Black, and Y. Feng (2025)
Chatgarment: garment estimation, generation and editing via large language models.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 2924–2934.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- A. Casado-Elvira, M. C. Trinidad, and D. Casas (2022)
Pergamo: personalized 3d garments from monocular video.
In Computer Graphics Forum,
Vol. 41, pp. 293–304.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- R. Chen, Y. Chen, N. Jiao, and K. Jia (2023)
Fantasia3d: disentangling geometry and appearance for high-quality text-to-3d content creation.
In Proceedings of the IEEE/CVF international conference on computer vision,
pp. 22246–22256.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- X. Chen, G. Wang, D. Zhu, X. Liang, P. Torr, and L. Lin (2022)
Structure-preserving 3d garment modeling with neural sewing machines.
Advances in Neural Information Processing Systems 35, pp. 15147–15159.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- W. Cong, H. Zhu, K. Wang, J. Lei, C. Stearns, Y. Cai, D. Wang, R. Ranjan, M. Feiszli, L. Guibas, et al. (2025)
Videolifter: lifting videos to 3d with fast hierarchical stereo alignment.
arXiv preprint arXiv:2501.01949.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- E. Corona, A. Pumarola, G. Alenya, G. Pons-Moll, and F. Moreno-Noguer (2021)
Smplicit: topology-aware generative model for clothed people.
In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition,
pp. 11875–11885.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- R. Daněřek, E. Dibra, C. Öztireli, R. Ziegler, and M. Gross (2017)
Deepgarment: 3d garment shape estimation from a single image.
In Computer Graphics Forum,
Vol. 36, pp. 269–280.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- E. Gundogdu, V. Constantin, A. Seifoddini, M. Dang, M. Salzmann, and P. Fua (2019)
Garnet: a two-stream network for fast and accurate 3d cloth draping.
In Proceedings of the IEEE/CVF international conference on computer vision,
pp. 8739–8748.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- J. Guo, J. Chen, W. Chen, Z. Sun, L. Li, B. Zhao, L. Zhu, X. Wang, and Q. Liu (2025)
GarmentX: autoregressive parametric representations for high-fidelity 3d garment generation.
arXiv preprint arXiv:2504.20409.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- K. He, K. Yao, Q. Zhang, J. Yu, L. Liu, and L. Xu (2024)
Dresscode: autoregressively sewing and generating garments from text guidance.
ACM Transactions on Graphics (TOG) 43 (4), pp. 1–13.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1),
[§3.2](https://arxiv.org/html/2602.16502v1#S3.SS2.SSS0.Px1.p1.19),
[§3.4](https://arxiv.org/html/2602.16502v1#S3.SS4.SSS0.Px2.p1.1).
- X. He, Z. Wu, X. Li, D. Kang, C. Zhang, J. Ye, L. Chen, X. Gao, H. Zhang, and H. Zhuang (2025)
Magicman: generative novel view synthesis of humans with 3d-aware diffusion and iterative refinement.
In Proceedings of the AAAI Conference on Artificial Intelligence,
Vol. 39, pp. 3437–3445.
Cited by: [§4.2](https://arxiv.org/html/2602.16502v1#S4.SS2.p1.1).
- Z. Huang, Y. Guo, H. Wang, R. Yi, L. Ma, Y. Cao, and L. Sheng (2025)
Mv-adapter: multi-view consistent image generation made easy.
In Proceedings of the IEEE/CVF International Conference on Computer Vision,
pp. 16377–16387.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- M. Korosteleva, T. L. Kesdogan, F. Kemper, S. Wenninger, J. Koller, Y. Zhang, M. Botsch, and O. Sorkine-Hornung (2024)
GarmentCodeData: a dataset of 3d made-to-measure garments with sewing patterns.
In European Conference on Computer Vision,
pp. 110–127.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- M. Korosteleva and S. Lee (2021a)
Generating datasets of 3d garments with sewing patterns.
arXiv preprint arXiv:2109.05633.
Cited by: [§3.2](https://arxiv.org/html/2602.16502v1#S3.SS2.SSS0.Px2.p1.3).
- M. Korosteleva and S. Lee (2021b)
Generating datasets of 3d garments with sewing patterns.
In Proceedings of the Neural Information Processing Systems Track on Datasets and Benchmarks, J. Vanschoren and S. Yeung (Eds.),
Vol. 1, pp. .
External Links: [Link](https://datasets-benchmarks-proceedings.neurips.cc/paper/2021/file/013d407166ec4fa56eb1e1f8cbe183b9-Paper-round1.pdf)
Cited by: [§5.1](https://arxiv.org/html/2602.16502v1#S5.SS1.SSS0.Px2.p1.1).
- M. Korosteleva and S. Lee (2022)
Neuraltailor: reconstructing sewing pattern structures from 3d point clouds of garments.
ACM Transactions on Graphics (TOG) 41 (4), pp. 1–16.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1),
[§1](https://arxiv.org/html/2602.16502v1#S1.p2.1),
[§3.2](https://arxiv.org/html/2602.16502v1#S3.SS2.SSS0.Px1.p1.19),
[Figure 4](https://arxiv.org/html/2602.16502v1#S5.F4).
- H. Kwon and S. Lee (2023)
DeepIron: predicting unwarped garment texture from a single image.
arXiv preprint arXiv:2310.15447.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- B. Li, X. Li, Y. Jiang, T. Xie, F. Gao, H. Wang, Y. Yang, and C. Jiang (2025a)
Garmentdreamer: 3dgs guided garment synthesis with diverse geometry and texture details.
In 2025 International Conference on 3D Vision (3DV),
pp. 1416–1426.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1),
[§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- H. Li, Z. Erkoç, L. Li, D. Sirigatti, V. Rosov, A. Dai, and M. Nießner (2025b)
MeshPad: interactive sketch-conditioned artist-reminiscent mesh generation and editing.
In Proceedings of the IEEE/CVF International Conference on Computer Vision,
pp. 16227–16237.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- M. Li, D. M. Kaufman, and C. Jiang (2020)
Codimensional incremental potential contact.
arXiv preprint arXiv:2012.04457.
Cited by: [§4.2](https://arxiv.org/html/2602.16502v1#S4.SS2.p1.1).
- R. Li, C. Cao, C. Dumery, Y. You, H. Li, and P. Fua (2025c)
Single view garment reconstruction using diffusion mapping via pattern coordinates.
In Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers,
pp. 1–11.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- R. Li, C. Dumery, B. Guillard, and P. Fua (2024a)
Garment recovery with shape and deformation priors.
In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition,
pp. 1586–1595.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- R. Li, B. Guillard, and P. Fua (2023)
Isp: multi-layered garment draping with implicit sewing patterns.
Advances in Neural Information Processing Systems 36, pp. 40294–40319.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- S. Li, R. Liu, C. Liu, Z. Wang, G. He, Y. Li, X. Jin, and H. Wang (2025d)
GarmageNet: a multimodal generative framework for sewing pattern design and generic garment modeling.
ACM Transactions on Graphics (TOG) 44 (6), pp. 1–23.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- X. Li, C. Yu, W. Du, Y. Jiang, T. Xie, Y. Chen, Y. Yang, and C. Jiang (2025e)
Dress-1-to-3: single image to simulation-ready 3d outfit with diffusion prior and differentiable physics.
ACM Transactions on Graphics (TOG) 44 (4), pp. 1–16.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p2.1),
[§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1),
[§4.2](https://arxiv.org/html/2602.16502v1#S4.SS2.p1.1).
- Y. Li, H. Chen, E. Larionov, N. Sarafianos, W. Matusik, and T. Stuyck (2024b)
Diffavatar: simulation-ready garment optimization with differentiable simulation.
In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition,
pp. 4368–4378.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p2.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- L. Liu, X. Xu, Z. Lin, J. Liang, and S. Yan (2023a)
Towards garment sewing pattern reconstruction from a single image.
ACM Transactions on Graphics (TOG) 42 (6), pp. 1–15.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- L. Liu, X. Xu, Z. Lin, J. Liang, and S. Yan (2023b)
Towards garment sewing pattern reconstruction from a single image.
ACM Transactions on Graphics (TOG) 42 (6), pp. 1–15.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p2.1),
[§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1),
[Figure 4](https://arxiv.org/html/2602.16502v1#S5.F4).
- S. Liu, Y. Cheng, Z. Chen, X. Ren, W. Zhu, L. Li, M. Bi, X. Yang, and Y. Yan (2025)
Multimodal latent diffusion model for complex sewing pattern generation.
In Proceedings of the IEEE/CVF International Conference on Computer Vision,
pp. 17640–17650.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p2.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- T. Liu and S. Li (2025)
Hybrid global-local representation with augmented spatial guidance for zero-shot referring image segmentation.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 29634–29643.
Cited by: [§3.3](https://arxiv.org/html/2602.16502v1#S3.SS3.SSS0.Px1.p1.5),
[§5.1](https://arxiv.org/html/2602.16502v1#S5.SS1.SSS0.Px1.p1.1).
- X. Liu, J. Li, and G. Lu (2023c)
Modeling realistic clothing from a single image under normal guide.
IEEE Transactions on Visualization and Computer Graphics 30 (7), pp. 3995–4007.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- G. Moon, H. Nam, T. Shiratori, and K. M. Lee (2022)
3d clothed human reconstruction in the wild.
In European conference on computer vision,
pp. 184–200.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- K. Nakayama, J. Ackermann, T. L. Kesdogan, Y. Zheng, M. Korosteleva, O. Sorkine-Hornung, L. J. Guibas, G. Yang, and G. Wetzstein (2025)
AIpparel: a multimodal foundation model for digital garments.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 8138–8149.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- C. Patel, Z. Liao, and G. Pons-Moll (2020)
Tailornet: predicting clothing in 3d as a function of human pose, shape and garment style.
In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition,
pp. 7365–7375.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- G. Pavlakos, V. Choutas, N. Ghorbani, T. Bolkart, A. A. Osman, D. Tzionas, and M. J. Black (2019)
Expressive body capture: 3d hands, face, and body from a single image.
In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition,
pp. 10975–10985.
Cited by: [§4.2](https://arxiv.org/html/2602.16502v1#S4.SS2.p1.1).
- B. Poole, A. Jain, J. T. Barron, and B. Mildenhall (2022)
Dreamfusion: text-to-3d using 2d diffusion.
arXiv preprint arXiv:2209.14988.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- G. Qian, J. Mai, A. Hamdi, J. Ren, A. Siarohin, B. Li, H. Lee, I. Skorokhodov, P. Wonka, S. Tulyakov, et al. (2023)
Magic123: one image to high-quality 3d object generation using both 2d and 3d diffusion priors.
arXiv preprint arXiv:2306.17843.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- N. Sarafianos, T. Stuyck, X. Xiang, Y. Li, J. Popovic, and R. Ranjan (2025)
Garment3dgen: 3d garment stylization and texture generation.
In 2025 International Conference on 3D Vision (3DV),
pp. 1382–1393.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- R. Shi, H. Chen, Z. Zhang, M. Liu, C. Xu, X. Wei, L. Chen, C. Zeng, and H. Su (2023)
Zero123++: a single image to consistent multi-view diffusion base model.
arXiv preprint arXiv:2310.15110.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- X. Su, S. You, F. Wang, C. Qian, C. Zhang, and C. Xu (2021)
Bcnet: searching for network width with bilaterally coupled network.
In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition,
pp. 2175–2184.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- Y. Tatsukawa, A. Qi, I. Shen, and T. Igarashi (2025)
GarmentImage: raster encoding of garment sewing patterns with diverse topologies.
In Proceedings of the Special Interest Group on Computer Graphics and Interactive Techniques Conference Conference Papers,
pp. 1–11.
Cited by: [§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).
- G. Team, R. Anil, S. Borgeaud, Y. Wu, J. Alayrac, J. Yu, R. Soricut, J. Schalkwyk, A. Dai, A. Hauth, et al. (2024)
Gemini: a family of highly capable multimodal models, 2024.
arXiv preprint arXiv:2312.11805 10.
Cited by: [§3.1](https://arxiv.org/html/2602.16502v1#S3.SS1.p1.11),
[§5.1](https://arxiv.org/html/2602.16502v1#S5.SS1.SSS0.Px1.p1.1).
- T. H. Team (2025a)
Hunyuan3D 2.0: scaling diffusion models for high resolution textured 3d assets generation.
External Links: 2501.12202
Cited by: [§3.3](https://arxiv.org/html/2602.16502v1#S3.SS3.SSS0.Px2.p1.4),
[§4.1](https://arxiv.org/html/2602.16502v1#S4.SS1.SSS0.Px1.p2.1),
[§5.1](https://arxiv.org/html/2602.16502v1#S5.SS1.SSS0.Px1.p1.1).
- T. H. Team (2025b)
Hunyuan3D 2.5: towards high-fidelity 3d assets generation with ultimate details.
External Links: 2506.16504,
[Link](https://arxiv.org/abs/2506.16504)
Cited by: [§3.1](https://arxiv.org/html/2602.16502v1#S3.SS1.p1.11).
- A. Vaswani, N. Shazeer, N. Parmar, J. Uszkoreit, L. Jones, A. N. Gomez, Ł. Kaiser, and I. Polosukhin (2017)
Attention is all you need.
Advances in neural information processing systems 30.
Cited by: [§3.4](https://arxiv.org/html/2602.16502v1#S3.SS4.SSS0.Px1.p1.9).
- V. Voleti, C. Yao, M. Boss, A. Letts, D. Pankratz, D. Tochilkin, C. Laforte, R. Rombach, and V. Jampani (2024)
Sv3d: novel multi-view synthesis and 3d generation from a single image using latent video diffusion.
In European Conference on Computer Vision,
pp. 439–457.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- J. Wang, M. Chen, N. Karaev, A. Vedaldi, C. Rupprecht, and D. Novotny (2025a)
Vggt: visual geometry grounded transformer.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 5294–5306.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- N. Wang, Y. Zhang, Z. Li, Y. Fu, W. Liu, and Y. Jiang (2018)
Pixel2mesh: generating 3d mesh models from single rgb images.
In Proceedings of the European conference on computer vision (ECCV),
pp. 52–67.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- R. Wang, Z. Cheng, Z. Lin, J. Ling, Y. Liu, Y. An, R. Xie, and L. Song (2025b)
SemanticGarment: semantic-controlled generation and editing of 3d gaussian garments.
In Proceedings of the 33rd ACM International Conference on Multimedia,
pp. 9793–9802.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- Z. Wang, C. Lu, Y. Wang, F. Bao, C. Li, H. Su, and J. Zhu (2023)
Prolificdreamer: high-fidelity and diverse text-to-3d generation with variational score distillation.
Advances in neural information processing systems 36, pp. 8406–8441.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- Z. Wu, I. Shen, and T. Igarashi (2025)
Real-time per-garment virtual try-on with temporal consistency for loose-fitting garments.
In Computer Graphics Forum,
Vol. 44, pp. e70272.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1).
- J. Xiang, Z. Lv, S. Xu, Y. Deng, R. Wang, B. Zhang, D. Chen, X. Tong, and J. Yang (2025)
Structured 3d latents for scalable and versatile 3d generation.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 21469–21480.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- H. Xie, H. Yao, X. Sun, S. Zhou, and S. Zhang (2019)
Pix2vox: context-aware 3d reconstruction from single and multi-view images.
In Proceedings of the IEEE/CVF international conference on computer vision,
pp. 2690–2698.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- Y. Xu, H. Tan, F. Luan, S. Bi, P. Wang, J. Li, Z. Shi, K. Sunkavalli, G. Wetzstein, Z. Xu, et al. (2023)
Dmv3d: denoising multi-view diffusion using 3d large reconstruction model.
arXiv preprint arXiv:2311.09217.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- X. Yang, D. Kukreja, D. Pinkus, A. Sagar, T. Fan, J. Park, S. Shin, J. Cao, J. Liu, N. Ugrinovic, M. Feiszli, J. Malik, P. Dollar, and K. Kitani (2025)
SAM 3d body: robust full-body human mesh recovery.
arXiv preprint; identifier to be added.
Cited by: [§3.1](https://arxiv.org/html/2602.16502v1#S3.SS1.p1.11),
[§3.3](https://arxiv.org/html/2602.16502v1#S3.SS3.SSS0.Px3.p1.2),
[§5.1](https://arxiv.org/html/2602.16502v1#S5.SS1.SSS0.Px1.p1.1).
- A. Yu, V. Ye, M. Tancik, and A. Kanazawa (2021)
Pixelnerf: neural radiance fields from one or few images.
In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition,
pp. 4578–4587.
Cited by: [§2.1](https://arxiv.org/html/2602.16502v1#S2.SS1.p1.1).
- Z. Yu, Z. Dou, X. Long, C. Lin, Z. Li, Y. Liu, N. Müller, T. Komura, M. Habermann, C. Theobalt, et al. (2024)
Surf-d: generating high-quality surfaces of arbitrary topologies using diffusion models.
In European Conference on Computer Vision,
pp. 419–438.
Cited by: [§2.2](https://arxiv.org/html/2602.16502v1#S2.SS2.p1.1).
- F. Zhou, R. Liu, C. Liu, G. He, Y. Li, X. Jin, and H. Wang (2025)
Design2GarmentCode: turning design concepts to tangible garments through program synthesis.
In Proceedings of the Computer Vision and Pattern Recognition Conference,
pp. 23712–23722.
Cited by: [§1](https://arxiv.org/html/2602.16502v1#S1.p1.1),
[§2.3](https://arxiv.org/html/2602.16502v1#S2.SS3.p1.1).