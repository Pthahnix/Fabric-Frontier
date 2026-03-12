<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - 2.1. Garment Sewing Pattern Reconstruction
    - 3D-based sewing pattern reconstruction.
    - Image-based sewing pattern reconstruction.
    - 3D garment mesh reconstruction from images
  - 2.2. Garment Datasets
  - 2.3. Textured Human Synthesis
- 3. SewFactory Dataset
  - 3.1. Garment Simulation
  - 3.2. Human Texture Synthesis
    - 3.2.1. Neural texture extraction.
    - 3.2.2. Human synthesis.
    - 3.2.3. Post-processing.
- 4. Sewing Pattern Reconstruction
  - 4.1. Architecture
    - 4.1.1. Visual encoder.
    - 4.1.2. Two-level Transformer decoder
    - 4.1.3. Stitch prediction.
  - 4.2. Loss Function
    - 4.2.1. Panel prediction loss
    - 4.2.2. SMPL-based regularization loss.
  - 4.3. Relationship with NeuralTailor
- 5. Experiments
  - 5.1. Implementation Details
    - Dataset.
    - Training.
  - 5.2. Comparison with the State of the Art
    - Evaluation metrics.
    - Quantitative evaluation.
    - Qualitative evaluation.
    - Comparing with NSM.
  - 5.3. Ablation Study
    - Effectiveness of the two-level architecture.
    - Effectiveness of the loss functions.
  - 5.4. Garment Reproduction and Editing
  - 5.5. Single-View Garment Mesh Reconstruction
  - 5.6. Generalization to Real Photos
    - Effectiveness of human texture synthesis.
    - Generalization to unseen topology.
    - Limitations.
- 6. Discussion and Future Work
- References

## Abstract

Abstract. Garment sewing pattern represents the intrinsic rest shape of a garment, and is the core for many applications like fashion design, virtual try-on, and digital avatars. In this work, we explore the challenging problem of recovering garment sewing patterns from daily photos for augmenting these applications. To solve the problem, we first synthesize a versatile dataset, named SewFactory, which consists of around 1M images and ground-truth sewing patterns for model training and quantitative evaluation. SewFactory covers a wide range of human poses, body shapes, and sewing patterns, and possesses realistic appearances thanks to the proposed human texture synthesis network. Then, we propose a two-level Transformer network called Sewformer, which significantly improves the sewing pattern prediction performance. Extensive experiments demonstrate that the proposed framework is effective in recovering sewing patterns and well generalizes to casually-taken human photos. Code, dataset, and pre-trained models are available at: https://sewformer.github.io .

## 1. Introduction

In this paper, we study the problem of estimating garment sewing patterns from a single RGB image (Fig. [1](#S0.F1)).
A garment sewing pattern (Bang et al., 2021; Korosteleva and Lee, 2022, 2021) is a collection of 2D polygons called panels that can be stitched together to form a garment.
It represents the intrinsic rest shape of a garment disentangled from the undesirable complicacy induced by extrinsic factors, such as external physical forces, collisions, and fabric properties.
Moreover, it is parametric and thus allows direct and interpretable control over the garment design.

Existing works on sewing pattern reconstruction mainly rely on rich 3D information as input, such as high-quality 3D scans (Bang et al., 2021) or point clouds (Korosteleva and Lee, 2022) which are not accessible for general users, limiting their applications in many practical scenarios.
In this work, we focus on a more challenging task: estimating sewing patterns from a single image, which can be obtained with only a regular camera or from monocular human photos on the Internet.
The estimated sewing pattern enables more accessible garment manipulation, such as clothes draping on novel body poses, and flexible editing of the shape of a captured garment, and further benefits many downstream applications in metaverse, such as virtual try-on, garment design, and avatar creation.
However, this problem is very challenging and is still less explored due to both the inherent ill-posedness of 2D-to-3D conversion and the complicacy of real-world conditions, such as varied camera views, challenging human poses, and occlusions.

Currently, among the few works in this direction, most of them adopt optimization-based approaches for sewing pattern reconstruction from images (Jeong et al., 2015; Yang et al., 2018).
Though achieving good results by imposing heuristic rules and priors, these methods are typically slow in inference due to the time-consuming optimization process, less user-friendly due to the difficulty of hyper-parameter tuning for different images, and susceptible to real-world inputs where the manually-designed rules could be violated.
Recently, Chen et al. (2022a) propose a deep neural network for recovering sewing patterns.
Nevertheless, this method does not consider the variations in garment types, human poses, and textures of real-world photos, and thereby cannot generalize well to in-the-wild data.

In this work, we target a data-driven framework for efficient and generalizable sewing pattern reconstruction using a single image.
There are two major challenges for building such a framework.
First, there is a lack of suitable training data.
Existing garment datasets either do not have sewing pattern annotations (Zhu et al., 2020) or are insufficiently diverse and realistic in terms of clothes appearances and human poses (Korosteleva and Lee, 2021).
To close this gap, we synthesize a new large-scale dataset, called SewFactory. SewFactory consists of one million image-and-sewing-pattern pairs with diverse garments under a wide range of human shapes and poses, facilitating more effective model training and evaluation.
In order to synthesize such a dataset, a common challenge faced by many recent datasets (Bertiche et al., 2020; Korosteleva and Lee, 2021) is that the rendered human body lacks photorealistic textures.
A simple solution to this problem is to directly apply pre-scanned human textures (Varol et al., 2017) onto human meshes. However, this approach often leads to low-quality appearances and artifacts due to poor scan quality, low human diversity, and 3D discontinuity in UV mapping.
To address this issue, we develop a novel neural network for textured human synthesis, which enhances our dataset by adding more realistic skin, faces, and hair.
Besides, SewFactory has abundant high-quality annotations, including depth maps, 3D human shapes and poses, garment meshes and textures, and 3D semantic segmentation labels, which could potentially benefit other tasks in Metaverse beyond this work.

The second challenge is that the data structures of sewing patterns are highly irregular and vary significantly across different samples. Specifically, different garments may consist of different numbers of panels, and different panels may be enclosed by different numbers of edges. Moreover, estimating the stitch information, i.e., how individual edges of panels are connected to each other, further complicates the problem.
To handle this issue, we propose a two-level Transformer network, called Sewformer, which aligns more closely with the data structure of sewing patterns. It separately learns the panel and edge representation in a hierarchical manner and thereby achieves higher-quality results than the existing baseline models.
In addition, we propose a new panel shape loss as well as an SMPL-based regularization loss for network training, which helps the proposed Sewformer learn from the large amount of training data more effectively.

To summarize, our contributions are as follows:

- •
We study the challenging problem of sewing pattern reconstruction from a single unconstrained human image.
We make early explorations to solve this problem and introduce Sewformer, a novel two-level Transformer network, which achieves high-quality results for sewing pattern reconstruction from an image.
- •
To facilitate effective model training and evaluation, we present SewFactory, a versatile and realistic dataset comprising approximately one million image-and-sewing-pattern pairs. This dataset offers diverse clothing styles and human poses, enabling improved model performance.
- •
To improve the quality of our data, we develop a novel method for human texture synthesis.
This method generates diverse and photorealistic human images under challenging poses, contributing to the realism and diversity of our dataset.
- •
We propose new loss functions that improve the training of Sewformer, further enhancing the quality of the reconstructed garment panels.

**Table 1. Comparison of the proposed SewFactory dataset with existing datasets. SewFactory is the first large-scale dataset that provides sewing pattern annotations along with high pose variation and realistic garment and human textures. “#Garment” denotes the number of garment instances. “Pose Var” indicates the level of pose variation exhibited in the dataset, with “None” denoting that all human is in a fixed T-pose. “G-Texture Var” and “H-Texture Var” represent garment texture and human texture variations, respectively.**
| Dataset | Real/Syn | #Garment | Pose Var | Sewing Pattern | G-Texture Var | H-Texture Var |
| --- | --- | --- | --- | --- | --- | --- |
| MGN (Bhatnagar et al., 2019) | Real | 712 | Low | ✘ | Low | Low |
| DeepFasion3D (Zhu et al., 2020) | Real | 563 | Low | ✘ | Low | None |
| 3DPeople (Pumarola et al., 2019) | Syn | 80 | High | ✘ | Low | Low |
| Cloth3D (Bertiche et al., 2020) | Syn | 11.3k | High | ✘ | High | Low |
| Wang et al. (Wang et al., 2018a) | Syn | 8k | None | ✔ | Low | None |
| Korosteleva and Lee (Korosteleva and Lee, 2021) | Syn | 22.5k | None | ✔ | Low | None |
| SewFactory | Syn | 19.1k | High | ✔ | High | High |

## 2. Related Work

### 2.1. Garment Sewing Pattern Reconstruction

Existing works for garment sewing pattern reconstruction can be roughly categorized into 3D-based (Hasler et al., 2007; Bang et al., 2021; Chen et al., 2015; Goto and Umetani, 2021; Korosteleva and Lee, 2022; Liu et al., 2018) and image-based approaches (Jeong et al., 2015; Yang et al., 2018; Chen et al., 2022a; Wang et al., 2018a), depending on their input type.

##### 3D-based sewing pattern reconstruction.

Early approaches in this direction use either template matching (Hasler et al., 2007; Chen et al., 2015) or surface flattening (Bang et al., 2021; Sharp and Crane, 2018) for sewing pattern reconstruction from point clouds or 3D meshes of dressed humans.
More recently, some researchers have explored data-driven methods for recovering sewing patterns from 3D data (Goto and Umetani, 2021; Korosteleva and Lee, 2022).
Goto and Umetani (2021) propose a deep convolutional neural network (CNN) (Isola et al., 2017) together with exponential map (Schmidt et al., 2006) for garment panel reconstruction.
Korosteleva and Lee (2022) propose NeuralTailor, which employs a hybrid network architecture for recovering the garment panels and stitching information from point clouds.
While these methods have achieved good results, their reliance on high-quality 3D input data limits their usage in many applications where 3D sensors are not accessible.

Figure: Figure 2. Annotations of the SewFactory dataset. We generate around one million RGB images where each is annotated with diverse ground-truth labels in the figure, supporting a wide variety of tasks in computer vision and graphics.
Refer to caption: /html/2311.04218/assets/x2.png

##### Image-based sewing pattern reconstruction.

Due to the intrinsic difficulties of sewing pattern reconstruction from 2D data, there are only a limited number of studies in this
area (Jeong et al., 2015; Yang et al., 2018; Wang et al., 2018a; Chen et al., 2022a).
Jeong et al. (2015) estimate sewing patterns from a single image by predicting the garment type and the primary body sizes via searching in a predefined database.
However, this method is less efficient due to the exhaustive search process and does not generalize well for poses that are not in the database.
Yang et al. (2018) recover sewing patterns by estimating panel parameters with iterative optimization (Kennedy and Eberhart, 1995).
Nevertheless, this method requires different templates for different garment types and relies on tedious preprocessing and registration steps, which can be both computationally intensive and error-prone.
Wang et al. (2018a) propose an encoder-decoder network for sewing pattern recovery from sketch images.
However, this approach requires training separate models for different garments, making it less practical for handling a wide range of garment types in real-world scenarios.
In addition, it is primarily designed for sketch images and cannot be easily applied to natural human photos.
More recently, Chen et al. (2022a) propose a CNN-based model that can predict garment panels from a single image for multiple garment types.
To handle the irregular data structure of various garment panels, this method employs PCA to simplify the panel data structure.
However, this approach is only designed for predefined panel groups, restricting the output garment space.
Moreover, due to the limitation of existing garment datasets, it only works well for human images in a standard T-pose, and its performance degenerates significantly for in-the-wild images that are captured under unconstrained poses.
Unlike the above methods, we propose a new model called Sewformer, which together with the proposed SewFactory dataset, can effectively handle the irregular structures of various garment panels without PCA compression and generate high-quality results for unconstrained human images with diverse shapes and poses.

##### 3D garment mesh reconstruction from images

This work is also related to recent works on image-based 3D garment mesh reconstruction (Wang et al., 2018b; Saito et al., 2019, 2020; Zhu et al., 2020; Patel et al., 2020; Tiwari and Bhowmick, 2021; Zhao et al., 2021; Moon et al., 2022).
These methods mainly train deep neural networks to directly regress the garment mesh from images, which can well reconstruct the coarse garment shape but struggle to produce realistic geometry details.
Moreover, most of these works suffer from over-smoothing artifacts in the areas invisible to the camera.
In contrast, our work provides a new paradigm for image-based 3D garment mesh reconstruction, which first recovers sewing patterns from the image and then generates 3D garment mesh with the recovered patterns using an off-the-shelf simulation engine.
This new paradigm is closer to the real physical process of constructing a posed garment. Therefore, it can achieve higher-quality 3D garment reconstruction with realistic geometry details and produce physically plausible results even for the occluded areas. This paradigm is also highly flexible for garment editing and character animation.

### 2.2. Garment Datasets

Garment data plays a key role in building data-driven models for sewing pattern recovery.
There are mainly two types of garment datasets: 3D scanning and physical simulation.
The first type is constructed by capturing real garments with 3D scanners (Zhang et al., 2017; Bhatnagar et al., 2019; Ma et al., 2020; Zhu et al., 2020; Tiwari et al., 2020).
While possessing realistic appearances and dynamics, these datasets are limited in size due to the high cost of 3D scanning.
For example,
a typical dataset of this kind,
DeepFasion3D (Zhu et al., 2020), comprises only 563 scanned garments, which is relatively small for training generalizable deep neural networks.
The second type of garment datasets uses physical simulation engines for data synthesis (Pumarola et al., 2019; Jiang et al., 2020; Bertiche et al., 2020; Hewitt et al., 2023), which overcomes the limitations of 3D-scan-based datasets and allows for the generation of larger amounts of garment data at a much lower cost.
Nevertheless, most of these datasets lack sewing pattern labels and thus are not suitable for our task.
Among the few existing datasets that do provide sewing patterns, (Narain et al., 2012) and (Wang et al., 2018a) do not have a diverse range of garment panels;
although (Korosteleva and Lee, 2021) provides various panel templates with different topologies, it does not consider the complex textures and materials of real-world garments and only simulates the garments on a fixed T-pose human model without a realistic human texture, which leads to a significant domain gap with real-world images, hindering the generalization ability of trained models.

To address these issues, we contribute a new synthetic garment dataset by sampling more sewing patterns and simulating the garments on a wide range of human shapes and poses.
We augment the dataset by adding realistic garment textures and material properties to mimic daily human photographs.
We also introduce a human texture synthesis network to generate diverse and realistic human appearances, which further improves the quality of our data.
The proposed dataset provides various high-quality ground-truth labels, such as sewing patterns, segmentation masks, 3D human shape and pose, and garment meshes, making it useful for a wide variety of applications in fashion research and industry.

### 2.3. Textured Human Synthesis

As mentioned above, we propose a human texture synthesis network to generate photorealistic human images for our garment dataset.
Since generating realistic textures plays an important role in many applications, such as human avatar creation, it has attracted great interest in recent years (Xian et al., 2018; Sarkar et al., 2020; Wang et al., 2019; Zhang et al., 2020; Lassner et al., 2017; Weng et al., 2020; Fu et al., 2022; Sarkar et al., 2021b; Albahar et al., 2021; Sarkar et al., 2021a; Xiang et al., 2022).

Some works for this problem aim to generate realistic textured human with clothed human images as exemplars based on the full-body sketch, segmentation mask, or pose-aware representations (Xian et al., 2018; Sarkar et al., 2020; Zhang et al., 2020; Albahar et al., 2021; Sarkar et al., 2021a). Meanwhile, another line of work synthesizes textured human from some pre-defined attributes (Weng et al., 2020) or from randomly sampled noise with GAN architecture (Fu et al., 2022; Sarkar et al., 2021b).

Recently, some works focus on generating texture maps that are 3D-aware or can be directly utilized on 3D human meshes (Alldieck et al., 2019; Bhatnagar et al., 2019; Huang et al., 2020; Lazova et al., 2019; Saito et al., 2020; Zhao et al., 2020; Han et al., 2019; Chaudhuri et al., 2021; Chen et al., 2022b; Grigorev et al., 2021; Yang et al., 2022; Xu and Loy, 2021). Among them, reconstruction-based methods (Alldieck et al., 2019; Bhatnagar et al., 2019; Huang et al., 2020; Lazova et al., 2019; Saito et al., 2020; Zhao et al., 2020) aim to predict the 3D geometry (e.g. normal) from RGB images. Some methods instead generate 3D-aware textures (Han et al., 2019) or full-body textured humans (Grigorev et al., 2021; Yang et al., 2022) by using garment templates or body shape and pose.

A major difference between these methods and our work is that to build the garment dataset, we are supposed to keep the simulated garments unchanged while synthesizing the human textures for the target pose.
This introduces additional challenges, as we need to generate humans with realistic skin and hair while maintaining challenging or even out-of-distribution poses without altering their corresponding garments. To fulfill these requirements, we devise a novel human texture synthesis network that introduces a learnable texture encoding, which facilitates effective texture extraction and warping.

## 3. SewFactory Dataset

We present a new dataset, SewFactory, for sewing pattern recovery from a single image.
A comprehensive comparison between SewFactory and other existing garment datasets can be found in Table [1](#S1.T1).
Notably, SewFactory possesses high pose variability and a diverse range of garments and human textures, which effectively closes the domain gap with real-world inputs.
Moreover, SewFactory provides abundant ground-truth labels as shown in Fig. [2](#S2.F2), which could potentially benefit many applications even beyond the task in this work. The whole pipeline for dataset generation is shown in Fig. [3](#S3.F3), which consists of two main steps: garment simulation and human texture synthesis.

### 3.1. Garment Simulation

In this step, we start by sampling a set of garment parameters, including sewing patterns, textures, and fabrics, as well as human body parameters. These parameters are then used to synthesize garments with a physical simulator (Choi and Ko, 2002).

Figure: Figure 3. Overview of the proposed data synthesis pipeline. We first generate high-quality garments under different human shapes and poses with physical simulation and then synthesize realistic human body textures with a novel neural network.
Refer to caption: /html/2311.04218/assets/x3.png

We use the templates in (Korosteleva and Lee, 2021) to generate the sewing patterns, which cover a wide range of garment shapes and topologies.
We first uniformly sample template parameters such as the sleeve length and hem width, and then generate the sewing pattern of each garment with the templates similar to (Korosteleva and Lee, 2021).
A sewing pattern is composed of two parts: a group of $N_{P}$ panels $\{P_{i}\}_{i=1}^{N_{P}}$ and their stitching information $S$.
Each panel $P_{i}$ is a closed 2D polygon encompassed by a list of $N_{i}$ edges $\{E_{i,j}\}_{j=1}^{N_{i}}$.
Each edge $E$ is a Bezier curve that can be represented by four scalars $x,y,c_{x},c_{y}$, where $(x,y)$ is the 2D vector from the start to the end of the edge, and $(c_{x},c_{y})$ is the control point of the Bezier curve defined in a local coordinate system.
In addition, each panel $P_{i}$ is associated with a 3D rotation $R_{i}\in\text{SO}(3)$ and a translation $T_{i}\in\mathbb{R}^{3}$, which describe the relative pose and position between the panel $P_{i}$ and the human body. $R_{i}$ and $T_{i}$ are used in the physical simulation process of cloth draping.
The stitching information $S$ is defined by pairs of edges $\{(i,j),(i^{\prime},j^{\prime})\}$ indicating that the $i$-th panel’s $j$-th edge is sewn with the $i^{\prime}$-th panel’s $j^{\prime}$-th edge.

To simulate a diverse range of garment data, we assign a unique set of material parameters to each sample, corresponding to a unique fabric object and textile image.
We utilize three commonly-used fabric presets, namely cotton, silk, and velvet, as the basic fabric types.
Each fabric encompasses various properties specific to that fabric type, and we introduce random perturbations to the property values to simulate garments with different levels of shininess, elasticity, and other fabric-specific characteristics.
In order to enhance the appearance diversity of the simulated garments, we curate a collection of approximately 600 textile images with free license.
These images serve as the source texture, to which we apply random augmentations, such as scaling and color jittering, to expand the range of visual appearances.

We use the Qualoth simulator (Choi and Ko, 2002) to simulate the garment dynamics.
During the simulation, we randomly pair up the top and bottom garments, such as T-shirts and pants, to form garment sets, and then drape each garment set onto a unique 3D human model.
The 3D human model is parameterized by SMPL (Loper et al., 2015), which uses two parameters $\beta\in\mathbb{R}^{10}$ and $\theta\in\mathbb{R}^{24\times 3}$ to control the 3D human shape and pose.
To ensure a sufficient amount of variation in pose, we sample 13.7k poses from the AMASS dataset (Mahmood et al., 2019) which are further interpolated into more poses using Maya (Autodesk, INC., 2019).
For each pose, we randomly sample a shape parameter $\beta$ from a uniform distribution within the range of [-1.5, 1.5].
This is in sharp contrast to existing datasets (Wang et al., 2018a; Korosteleva and Lee, 2021) that only consider garments under a fixed T-pose template, leading to a significant domain gap with real-world photos.

For each simulated garment, we render 24 views from cameras uniformly distributed around the human body using Arnold renderer (Georgiev et al., 2018).
In some rare cases, the simulator produces inappropriate simulation results, such as with wrong sizes or self-intersection, which are manually removed.
Overall, the garment simulation leads to around one million RGB images with paired sewing pattern labels, where 85k images are used for testing, and the remaining is used for training.
Our data splitting ensures no garment or pose is repeated in training and testing.

### 3.2. Human Texture Synthesis

While the proposed simulation system is able to produce high-quality garments and sewing patterns, the generated images still suffer from an important shortcoming that the rendered human body lacks photorealistic textures (Fig. [4](#S3.F4) (a)).
Note that this is a common challenge faced by many recent datasets (Bertiche et al., 2020; Korosteleva and Lee, 2021).
A simple solution to this problem is to directly apply pre-scanned human textures, e.g., SURREAL (Varol et al., 2017), onto human meshes.
However, this approach often leads to low-quality results (Fig. [4](#S3.F4) (b)) due to noise and artifacts in scanning, limited human diversity, and 3D discontinuities in UV mapping.
To address this issue, we develop a deep neural network for human texture synthesis, which enhances our dataset by adding more realistic skin, faces, and hair.
As shown in Fig. [4](#S3.F4) (c), this network allows us to greatly improve the realism and overall quality of the generated images.
Instead of creating human textures from scratch, we utilize images of real humans (Liu et al., 2016) as a reference. This simplifies our task by reducing it to transferring the texture of a real human (Fig. [4](#S3.F4) (e)) to the target pose (Fig. [4](#S3.F4) (f)).
Specifically, given a reference image $R_{\text{img}}\in\mathbb{R}^{3\times H\times W}$ where $H$ and $W$ are the height and width of the image, we aim to extract the appearance information of $R_{\text{img}}$ and then apply it to the target pose $T_{\text{pose}}\in\mathbb{R}^{3\times H\times W}$.
We use Densepose (Güler et al., 2018) to represent the target pose, as we find it better represents the semantic information of different pixels of the human body.

One straightforward approach to this texture transfer task is to directly concatenate $R_{\text{img}}$ and $T_{\text{pose}}$ and then feed the concatenated input into a deep CNN to generate the desired human image.
However, this method has a major drawback in that it cannot accurately preserve the target pose, resulting in artifacts due to mismatch with the simulated garment.
Moreover, it tends to overfit training poses and degenerates severely on out-of-distribution poses in test data.
To address this issue, we propose to first extract the texture map of the human body and then warp the texture to the target pose.
An overview of the proposed framework for textured human synthesis is shown in Fig. [5](#S3.F5), which is comprised of three stages as below.

Figure: Figure 4. Comparison of different human textures. (a) Original simulation output that does not have human texture; (b) textured human by using pre-scanned SURREAL texture (Varol et al., 2017); (c) our result; (d) our result without diffusion editing. Instead of creating human textures from scratch, we synthesize textured human by transferring the texture of a reference image (e) to the target pose (f).
Refer to caption: /html/2311.04218/assets/x4.png

#### 3.2.1. Neural texture extraction.

At the core of the texture extraction stage is the texture extractor in Fig. [5](#S3.F5).
It maps a texture encoding $T_{\text{enc}}\in\mathbb{R}^{D\times N_{\text{p}}\times H_{\text{tex}}\times W_{\text{tex}}}$ to a texture map $T_{\text{tex}}\in\mathbb{R}^{D\times N_{\text{p}}\times H_{\text{tex}}\times W_{\text{tex}}}$ by aggregating the information from the reference image $R_{\text{img}}$, where $H_{\text{tex}},W_{\text{tex}},D$ are the height, width, and number of channels of the texture map, and $N_{\text{p}}$ is the number of body parts defined in Densepose (Güler et al., 2018).
$T_{\text{tex}}$ can be seen as a generalization of the UV map in (Xu and Loy, 2021), which describes the 3D textures of different body parts.
$T_{\text{enc}}$ is a learnable encoding tensor that is randomly initialized and shared across different samples.
Similar to (Xu and Loy, 2021), we first use a deep CNN encoder to convert $R_{\text{img}}$ into feature space. Then texture extractor exploits the cross attention mechanism to non-locally distribute the information of $R_{\text{img}}$ into $T_{\text{tex}}$.
Next, we warp the learned texture features $T_{\text{tex}}$ to the target pose $T_{\text{pose}}$ with bilinear sampling, which generates the warped features $T_{\text{warp}}\in\mathbb{R}^{D\times H\times W}$.
Our neural texture extraction approach ensures the synthesized human body always conforms to the target pose, resulting in a proper fit for the simulated garment.
In addition, as the texture extractor is decoupled from the target pose, our method is less susceptible to out-of-distribution test poses.

Figure: Figure 5. Overview of the proposed framework for human texture synthesis. First, we use a shared texture encoding $T_{\text{enc}}$ to extract texture features $T_{\text{tex}}$ from the reference image $R_{\text{img}}$ and then warp these features to the target pose $T_{\text{pose}}$. Next, we generate textured human $T_{\text{gen}}$ by first mapping the warped features $T_{\text{warp}}$ to RGB human textures $T_{\text{human}}$ and then fusing $T_{\text{human}}$ with the simulated garment $T_{\text{garment}}$ with mask fusion. Finally, we employ a diffusion editing module to further refine the synthesized result.
Refer to caption: /html/2311.04218/assets/x5.png

#### 3.2.2. Human synthesis.

While one can use $T_{\text{warp}}$ as the final output (by setting $D=3$), it often leads to low-quality head regions as the Densepose is not capable of precisely depicting the human head.
Instead, we generate the textured human $T_{\text{human}}$ with another deep CNN (“Texture synthesis” in Fig. [5](#S3.F5)) under the guidance of the head part of the reference image, denoted by $R_{\text{head}}$.
We adopt the U-Net architecture (Ronneberger et al., 2015) for the texture synthesis CNN, and the guidance information of $R_{\text{head}}$ is injected with the adaptive normalization of StyleGAN (Karras et al., 2019).
With the synthesized human body $T_{\text{human}}$, we can generate the clothed human image with mask fusion:

$$ $\displaystyle T_{\text{gen}}=T_{\text{human}}\cdot(1-\text{Mask})+T_{\text{garment}}\cdot\text{Mask},$ $$

where $T_{\text{garment}}$ and Mask are the garment texture and mask obtained from the simulation system.

We train our network with a combination of pixel-wise loss, perceptual loss (Johnson et al., 2016), feature-matching loss (Xu et al., 2017), and adversarial loss (Goodfellow et al., 2020).

#### 3.2.3. Post-processing.

As shown in Fig. [4](#S3.F4) (d), the direct output $T_{\text{gen}}$ from the human synthesis network still contains a considerable amount of artifacts and distortions.
To further improve the quality of the generated human, we employ diffusion models (Rombach et al., 2022) as a powerful image prior for post-processing.
Specifically, we follow the SDEdit framework (Meng et al., 2021) to refine the human images. We first perturb the synthesized image $T_{\text{gen}}$ with a moderate ratio of Gaussian noise and then progressively remove the noise with a denoising network, which effectively improves the realism of the final output $T_{\text{out}}$ (Fig. [4](#S3.F4) (c)) and closes the gap with real human photos.
Eventually, the proposed data generation pipeline enables highly diversified results with various poses and realistic appearances as shown in Fig. [6](#S3.F6).

Figure: Figure 6. Example results of human texture synthesis. The proposed algorithm generalizes well to various challenging poses in the black boxes.
Refer to caption: /html/2311.04218/assets/x6.png

## 4. Sewing Pattern Reconstruction

Figure: Figure 7. Overview of the proposed Sewformer. We first send the input image into a visual encoder (a) to generate a sequence of visual tokens. Then the visual tokens are fed into a two-level Transformer decoder (b) to produce panel-level and edge-level feature tokens, which are subsequently used to recover the garment panels. The edge features are also used to estimate the stitching relations via the stitch prediction module (c).
Refer to caption: /html/2311.04218/assets/x7.png

As introduced in Section [3.1](#S3.SS1), the sewing pattern in our SewFactory dataset has a highly irregular data structure, making it unsuitable for the commonly-used deep CNNs that are primarily designed for regular data structures.
Instead, we propose a new model, called Sewformer, to better accommodate this data. We also propose new loss functions to facilitate training of Sewformer.

### 4.1. Architecture

An overview of Sewformer is shown in Fig. [7](#S4.F7).
It consists of three main components: (a) a visual encoder to learn sequential visual representations from the input image, (b) a two-level Transformer decoder to obtain the sewing pattern in a hierarchical manner, and (c) a stitch prediction module that recovers how different panels are stitched together to form a garment.

One of the key designs of our work is the two-level structure of the Transformer decoder. It provides a simple yet effective way to handle garment sewing patterns. This design is novel and has not been previously applied to the task of garment reconstruction.

#### 4.1.1. Visual encoder.

To use Transformers for sewing pattern reconstruction, we employ the visual encoder to convert the input image into sequential data.
Specifically, we first generate a low-resolution feature map $F\in\mathbb{R}^{C\times H_{F}\times W_{F}}$ from the input image with a CNN backbone (ResNet-50 (He et al., 2016) in our experiments).
Then we serialize $F$ by reshaping it into ${C\times H_{F}W_{F}}$ where the spatial dimensions are flattened in 1D.
These serialized features are subsequently processed by a Transformer encoder to learn the visual tokens $F_{\text{vis}}\in\mathbb{R}^{C\times H_{F}W_{F}}$, where each encoder layer consists of a standard multi-head self-attention module and an MLP similar to (Vaswani et al., 2017).

#### 4.1.2. Two-level Transformer decoder

Based on the learned visual tokens $F_{\text{vis}}$,
we propose a two-level Transformer decoder to recover the garment panels $\{P_{i}\}_{i=1}^{N_{P}}$ that are introduced in Section [3.1](#S3.SS1).
As shown in Fig. [7](#S4.F7), the first level (panel decoder) is designed to extract the overall information of the panels; the second level (edge decoder) is dedicated to learning the specific shapes of the panels by recovering the edge information.
As will be shown in Section [5](#S5), compared to normal Transformers that only have a one-level decoder, the proposed Sewformer considers the two-level (panel and edge) characteristics of sewing patterns and is able to more effectively recover the garments in a coarse-to-fine manner.

For the panel decoder, we first randomly initialize $N_{P}$ panel queries $\{Q_{\text{P}}^{i}\}_{i=1}^{N_{P}}$ with Gaussian distribution. These panel queries are trained using backpropagation along with the whole model.
We predict the panel tokens $\{F_{P}^{i}\}_{i=1}^{{N_{P}}}$ by applying a multi-head cross-attention layer between the queries $Q_{P}^{i}$ and the visual tokens $F_{\text{vis}}$:

$$ (1) $\displaystyle{F_{P}^{i}}=\text{CrossAttention}(Q_{\text{P}}^{i},F_{\text{vis}}).$ $$

Then we can use an MLP to predict the 3D rotation and translation of the panels, i.e., $R_{i},T_{i}=\text{MLP}({F_{P}^{i}})$.

Similar to the panel decoder, we initialize a set of random query embeddings for all the panel edges in the edge decoder which are learned in training.
We use a maximum number of panel and edge queries and remove invalid predictions during inference (edges with lengths near zero and panels with fewer than three edges).
Then we combine each edge query with its corresponding panel feature using element-wise sum and pass the combined embeddings into an MLP to obtain the final edge queries.
In this way, the edge queries belonging to the same panel share the same panel feature, which facilitates producing consistent edges.

Furthermore, the coarse-to-fine learning process, which progresses from panel to edge, effectively alleviates the difficulties in training, while directly learning a large number of edges could be overwhelming and lead to undesirable local minimums.

Similar to Eq. [1](#S4.E1), we feed the edge queries and visual tokens into a cross-attention layer to generate each edge token $F_{E}^{ij}$ (corresponding to the $j$-th edge of the $i$-th panel).
The edge tokens are then used in an MLP to predict the Bezier edge parameter $E_{i,j}=\text{MLP}({F_{E}^{ij}})$.

#### 4.1.3. Stitch prediction.

Our stitch prediction module is similar to that of NeuralTailor (Korosteleva and Lee, 2022).
As introduced in Section [3.1](#S3.SS1), the stitches are defined by pairs of edges from different panels.
Since stitched two edges typically have similar features, such as orientations and spatial locations, we predict whether two edges form a stitch based on the similarity between the edge features.
As shown in Fig. [7](#S4.F7), we first construct the similarity matrix between all pairs of edges in the predicted sewing pattern using the learned edge tokens as stitch tags and then obtain the stitch predictions by iteratively finding the maximum values in the matrix.
Specifically, since the similarity matrix is symmetric, we first remove its lower triangular part. We identify the maximum value of the remaining entries and then eliminate the corresponding row and column. This process is iterated until all the stitches are determined.

### 4.2. Loss Function

Our training objective is composed of three parts: a panel prediction loss, a stitch prediction loss, and an SMPL-based regularization term:

$$ (2) $\mathcal{L}_{\text{total}}=\lambda_{1}\mathcal{L}_{\text{panel}}+\lambda_{2}\mathcal{L}_{\text{stitch}}+\lambda_{3}\mathcal{L}_{\text{SMPL}},$ $$

where $\lambda_{1},\lambda_{2},\lambda_{3}$ are hyperparameters to balance each term. We use a stitch prediction loss $\mathcal{L}_{\text{stitch}}$ similar to (Korosteleva and Lee, 2022), and the other two terms are explained below.

#### 4.2.1. Panel prediction loss

The panel prediction loss consists of three terms: 1) the shape loss $\mathcal{L}_{\text{shape}}$ to encourage high-fidelity shapes of the reconstructed panels; 2) the loop loss $\mathcal{L}_{\text{loop}}$ to enforce that the edges of a panel form a closed loop; 3) the rotation and translation loss $\mathcal{L}_{\text{RT}}$ to encourage accurate 3D rotation and translation predictions:

$$ (3) $\mathcal{L}_{\text{panel}}=\mathcal{L}_{\text{shape}}+\mathcal{L}_{\text{loop}}+\mathcal{L}_{\text{RT}},$ $$

where $\mathcal{L}_{\text{loop}}$ and $\mathcal{L}_{\text{RT}}$ are defined the same way as NeuralTailor (Korosteleva and Lee, 2022).

For the shape loss, a straightforward choice is to use the one in NeuralTailor (Korosteleva and Lee, 2022) as well,
which directly penalizes the L2 distance between the predicted edge and the ground-truth edge (solid blue lines in Fig. [8](#S4.F8)(a)). Nevertheless, we find that this is not always an adequate measure of shape discrepancy between different panels.
For instance, despite the shape of Prediction-1 in Fig. [8](#S4.F8)(b) being closer to the ground truth in Fig. [8](#S4.F8)(a) than Prediction-2 in Fig. [8](#S4.F8)(c), the two predictions yield the same value for the per-edge loss $\mathcal{L}_{\text{shape-NT}}$.

An important cause for this problem is that the per-edge loss only provides 1D comparisons (lines) between the prediction and the ground truth, resulting in sparse and implicit supervision of the 2D shapes. To explicitly enforce shape similarity in 2D, a better solution is to convert the panel edges into binary 2D masks and penalize the discrepancy between these masks. However, this conversion involves rasterization operations that are non-differentiable, resulting in difficulties in training.

To address this issue, we propose a novel shape loss $\mathcal{L}_{\text{shape}}$ to approximate the 2D mask loss, as illustrated in Fig. [8](#S4.F8)(d).
We start by collecting a set of vertices on the panel edges and sampling support vectors on the panel by connecting each pair of vertices. Our shape loss $\mathcal{L}_{\text{shape}}$ is then defined as the L2 error between the support vectors in the predicted panels and those in the ground-truth panels.
$\mathcal{L}_{\text{shape}}$ can be seen as a densified version of the per-edge loss $\mathcal{L}_{\text{shape-NT}}$, which better encourages 2D shape similarity with the ground truth.
As expected, the more support vectors that are sampled, the better our shape loss approximates the 2D mask loss. In our implementation, we use the endpoints and midpoints of the panel edges as the vertices for simplicity.

To more intuitively understand why sampling more vectors on the panel leads to a better shape loss, we provide a toy example of two edge vectors on the panel, i.e., $v_{1}$ and $v_{2}$ in Fig. [8](#S4.F8)(b).
Denoting the ground-truth vectors as $\hat{v}_{1}$ and $\hat{v}_{2}$, the per-edge loss could be written as:

$$ (4) $\displaystyle\|v_{1}-\hat{v}_{1}\|+\|v_{2}-\hat{v}_{2}\|=\|\Delta v_{1}\|+\|\Delta v_{2}\|,$ $$

where we define $\Delta v=v-\hat{v}$, and $\|\cdot\|$ represents the L2 norm.
Correspondingly, the proposed $\mathcal{L}_{\text{shape}}$ becomes:

$$ $\displaystyle\|v_{1}-\hat{v}_{1}\|+\|v_{2}-\hat{v}_{2}\|+\|(v_{1}+v_{2})-(\hat{v}_{1}+\hat{v}_{2})\|$ $\displaystyle=$ $\displaystyle\|\Delta v_{1}\|+\|\Delta v_{2}\|+\|\Delta v_{1}+\Delta v_{2}\|$ (5) $\displaystyle=$ $\displaystyle(1+\cos\alpha_{1})\|\Delta v_{1}\|+(1+\cos\alpha_{2})\|\Delta v_{2}\|,$ $$

where $v_{1}+v_{2}$ is the newly-added support vector in Fig. [8](#S4.F8)(b), and the angles $\alpha_{1}$ and $\alpha_{2}$ are defined in Fig. [8](#S4.F8)(e).
By comparing Eq. [4](#S4.E4) and [4.2.1](#S4.Ex2), we can see that the proportion between the two errors $\|\Delta v_{1}\|$ and $\|\Delta v_{2}\|$ are adjusted by the proposed shape loss.
Suppose that the prediction of $v_{1}$ has a larger error than $v_{2}$, i.e., $\|\Delta v_{1}\|>\|\Delta v_{2}\|$, then we have $(1+\cos\alpha_{1})/(1+\cos\alpha_{2})>1$, implying that larger errors are more heavily penalized in Eq. [4.2.1](#S4.Ex2).
In other words, $\mathcal{L}_{\text{shape}}$ encourages more evenly distributed errors among all edges, and thereby the prediction in Fig. [8](#S4.F8)(b) is preferred over Fig. [8](#S4.F8)(c).

For simplicity of the explanation, we assume the predicted edges are lines instead of Bezier curves in Eq. [4](#S4.E4). However, the underlying concept remains valid, as curves can be effectively approximated by piece-wise lines.

Figure: Figure 8. Illustration of the proposed panel shape loss. (a) depicts a ground truth panel. (b) Each edge of Prediction-1 is slightly shorter than the corresponding edge in the ground truth. (c) While most edges of Prediction-2 match the ground truth exactly, the two right-side edges exhibit significant errors. Overall, (b) and (c) have the same per-edge loss $\mathcal{L}_{\text{shape-NT}}$ (Korosteleva and Lee, 2022). (d) shows the support vectors connecting vertices on the edges, including the endpoints (green) and midpoints (orange). We only show the support vectors emanating from the dark green vertex and omit others for clarity. (e) provides an intuitive analysis of $\mathcal{L}_{\text{shape}}$, showing how it encourages more evenly distributed errors among edges of the panel.
Refer to caption: /html/2311.04218/assets/x8.png

#### 4.2.2. SMPL-based regularization loss.

To further improve the estimation of 3D garment panels, a good understanding of the 3D human pose in the input image could be beneficial.
As the garment panels are typically closely related to the shape and movement of the body, the 3D human pose that describes the position and orientation of body parts can provide vital information to infer the shape and location of the garment panels in 3D space, even when they are partially visible or occluded in the input image.

Motivated by this idea, we introduce an SMPL-based regularization loss term in Eq. [2](#S4.E2) to guide the training process of Sewformer.
Specifically, we add an extra set of pose queries to the panel queries in Fig. [7](#S4.F7)(b), which leads to an additional output of 3D human pose $\theta$ after the panel decoder.
Then the regularization term $\mathcal{L}_{\text{SMPL}}$ is defined as the mean squared error between the predicted 3D human pose and the ground truth.
As the learned pose features are adaptively blended into panel tokens by the attention mechanism in the panel decoder, $\mathcal{L}_{\text{SMPL}}$ essentially facilitates garment panel reconstruction with human pose information.
Note that we did not supervise the human shape $\beta$ as we empirically find no benefits in our experiments.
Thanks to the abundant labels provided by the SewFactory dataset, we can easily apply the SMPL regularization term in a supervised manner.

Figure: Figure 9. Qualitative evaluation of panel predictions by different methods on our dataset. For each method, the two columns represent the front view and back view results, respectively. Major errors of the baseline approaches are highlighted with red edges. In the last row, the display order for the 4-panel skirt is left-front-right-back. Please see the caption of Table [2](#S4.T2) for the explanations of the baseline methods.
Refer to caption: /html/2311.04218/assets/x9.png

### 4.3. Relationship with NeuralTailor

The proposed Sewformer is related to NeuralTailor (Korosteleva and Lee, 2022), as both designs share similar elements such as the loop and RT losses in Eq. [3](#S4.E3).
Nevertheless, Sewformer introduces several fundamental improvements and contributions that distinguish it from NeuralTailor.
First, Sewformer utilizes a new two-level Transformer model, which effectively handles the complex data structure of sewing patterns.
This design is a significant departure from the hybrid architecture of EdgeCNN, attention-MLP, and LSTM used in NeuralTailor.
While being much simpler, it results in better accuracy in high-quality sewing pattern reconstruction.
Second, we introduce two novel loss functions, $\mathcal{L}_{\text{shape}}$ and $\mathcal{L}_{\text{SMPL}}$, which significantly improve the training process.
In addition, different from prior methods (Korosteleva and Lee, 2022; Chen et al., 2022a) that only handle one garment at a time (upper or lower clothes), the proposed Sewformer is capable of predicting the panels for the entire clothing set in a single model run.
To separate the recovered panels into upper and lower garments, we leverage the predicted stitching relations and group connected panels into the same clothes piece.
Lastly, empowered by the SewFactory dataset, our proposed algorithm is capable of effectively handling casual human photos captured in unconstrained poses, while NeuralTailor is limited to T-pose inputs, restricting its applicability in real-world scenarios.

**Table 2. Quantitative evaluation on the SewFactory dataset. The evaluation metrics are introduced in Section [5.2](#S5.SS2). Sewformer^† is a variant of our model with a one-level Transformer decoder. NeuralTailor^∗ is the NeuralTailor model (Korosteleva and Lee, 2022) trained with our proposed loss functions. $\uparrow$: the higher the better; $\downarrow$: the lower the better.**
| Model | Panel L2 $\downarrow$ | Rot L2 $\downarrow$ | Trans L2 $\downarrow$ | #Panel $\uparrow$ | #Edges $\uparrow$ | Precision $\uparrow$ | Recall $\uparrow$ | F1 score $\uparrow$ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Sewformer | 3.57 | 0.0205 | 0.693 | 88.7% | 97.5% | 96.1% | 95.4% | 95.7% |
| Sewformer^† | 3.91 | 0.0322 | 0.979 | $87.5\%$ | $95.6\%$ | $82.8\%$ | $98.9\%$ | $90.1\%$ |
| NeuralTailor^∗ | 4.15 | 0.0347 | 0.995 | $83.8\%$ | $97.5\%$ | $76.8\%$ | 99.6% | $86.7\%$ |
| NeuralTailor | 4.41 | 0.0300 | 1.050 | $83.6\%$ | 97.8% | $81.5\%$ | $87.8\%$ | $84.5\%$ |

## 5. Experiments

We first describe the implementation details of the proposed Sewformer. Then we provide qualitative and quantitative evaluations of our algorithm on both synthetic and real-world images.

### 5.1. Implementation Details

##### Dataset.

We use the SewFactory dataset to train and evaluate our proposed Sewformer.
We group panels of different garment types based on their spatial attributes, such as top front, top back, sleeve left front, etc., resulting in 24 semantic panel classes.
For efficient training, we align all garment parameters and ensure that they have the same dimension by applying zero padding.

##### Training.

During training, we apply random rotation, affine, and perspective augmentations to the input images, followed by resizing to a fixed size of $384\times 384$. We use the AdamW optimizer. We set the initial learning rate of the Transformer to $10^{-4}$, the initial learning rate of the CNN backbone to $10^{-5}$, and the weight decay to $10^{-4}$. The backbone is initialized with pre-trained weights from ImageNet, while the rest of the model is initialized randomly. We train the models for 40 epochs using 8 A100 GPUs with a total batch size of 512. To balance the different loss terms in the objective function (Eq. [2](#S4.E2)), we set the hyperparameters as $\lambda_{1}=10$, $\lambda_{2}=0.5$, and $\lambda_{3}=1$.

### 5.2. Comparison with the State of the Art

We compare the proposed algorithm against NeuralTailor (Korosteleva and Lee, 2022), the state-of-the-art method for recovering garment sewing patterns.
Since NeuralTailor is designed for 3D point clouds, we adapt it to our task by replacing the original graph-based encoder with a ResNet-50 architecture for feature extraction. The extracted image features are spatially flattened and treated as point features in the subsequent modules of NeuralTailor.
As NeuralTailor is originally trained on the dataset of (Korosteleva and Lee, 2021), it is only suitable for fixed T-pose garments and cannot handle diverse human poses captured in everyday photos.
For a more comprehensive comparison, we retrain NeuralTailor on our proposed SewFactory dataset.

##### Evaluation metrics.

We use the same evaluation metrics as (Korosteleva and Lee, 2022):
1) Panel L2: the L2 distance between the edge parameters of the predicted and ground-truth panels, measuring the quality of shape predictions;
2) Rot L2 and Trans L2: the L2 error of the predicted rotations $R$ and translations $T$ compared to their ground truth;
3) #Panel: the accuracy of the predicted number of panels within each garment pattern;
4) #Edges: the accuracy of the number of edges within each correctly-predicted panel;
5) the precision, recall, and F1 score of the stitches, evaluating the quality of recovered stitching relations.

##### Quantitative evaluation.

As shown in Table [2](#S4.T2), our Sewformer demonstrates superior performance compared to NeuralTailor across multiple evaluation metrics.
In particular, our method achieves a relative decrease of 19% in Panel L2 error, a relative decrease of 32% in Rot L2 error, a relative decrease of 34% in Trans L2 error, an absolute increase of 5.1% in #Panel accuracy, and an absolute increase of 11.2% in F1 score, showing the effectiveness of our algorithm.
Meanwhile, our method exhibits a slightly lower accuracy in #Edges compared to NeuralTailor.
This discrepancy arises due to the fact that our approach well recovers more panels than NeuralTailor, including panels that consist of particularly challenging edges. Consequently, these challenging edges contribute to the increased complexity of edge estimation in our #Edges results.

**Table 3. Effectiveness of the proposed loss functions. “w/o $\mathcal{L}_{\text{shape}}$” represents the model trained with $\mathcal{L}_{\text{shape-NT}}$.**
| Model | Panel L2 $\downarrow$ | Rot L2 $\downarrow$ | Trans L2 $\downarrow$ | #Panel $\uparrow$ | #Edges $\uparrow$ |
| --- | --- | --- | --- | --- | --- |
| full model | 3.57 | 0.0205 | 0.693 | 88.7% | 97.5% |
| w/o $\mathcal{L}_{\text{shape}}$ | 3.71 | 0.0260 | 0.966 | 88.2% | 97.8% |
| w/o $\mathcal{L}_{\text{SMPL}}$ | 3.63 | 0.0243 | 0.897 | 86.1% | 97.9% |

Furthermore, it is worth mentioning that Sewformer and NeuralTailor are trained using different loss functions. To emphasize the efficacy of the architecture of Sewformer, we also retrain NeuralTailor using our proposed loss functions (denoted as NeuralTailor^∗ in Table [2](#S4.T2)). Although NeuralTailor^∗ demonstrates improvements across most metrics compared to the baseline, it still clearly falls short of the performance of Sewformer.

##### Qualitative evaluation.

We present qualitative evaluations in Fig. [9](#S4.F9).
It is evident that our proposed model outperforms NeuralTailor by a substantial margin. Particularly, the results obtained from NeuralTailor exhibit significant deficiencies in terms of fidelity and accuracy. For instance, the skirt appears distorted, and the waistband takes on an irregular polygonal form, deviating from the ground-truth rectangular shape.
In contrast, Sewformer produces more precise details, such as the length of the skirts and pants.

Figure: Figure 10. Comparison with NSM (Chen et al., 2022a). The proposed algorithm achieves higher-quality garment reconstruction.
Refer to caption: /html/2311.04218/assets/x10.png

##### Comparing with NSM.

We further compare the proposed Sewformer with NSM (Chen et al., 2022a).
Since the code for NSM is not publicly available, we directly use the reconstructed garment mesh provided by the original authors for comparison.
As shown in Fig. [10](#S5.F10), our method produces high-quality results, while the results of NSM suffer from low fidelity and artifacts.
Moreover, these artifacts make it problematic to simulate the garment of NSM for different poses, while our results can be conveniently manipulated and edited as elaborated in Section [5.4](#S5.SS4).

### 5.3. Ablation Study

##### Effectiveness of the two-level architecture.

In Section [4.1](#S4.SS1), we propose a two-level Transformer decoder to handle the hierarchical structure of sewing patterns.
To investigate the impact of this design, we train a variant of the Sewformer with a one-level decoder (denoted as Sewformer^†), which relies on the panel tokens to query the garment information from the visual encoder and predicts the final edge vectors with a subsequent MLP.
As shown in Table [2](#S4.T2), while Sewformer^† gives better results than NeuralTailor, there is still a clear performance gap between Sewformer^† and Sewformer, highlighting the effectiveness of the two-level design.
Further, the visual examples in Fig. [9](#S4.F9) also support this observation,
where the results of Sewformer^† lack important details, especially in panel corners. In contrast, the proposed model consistently generates more accurate results, demonstrating the crucial role of the two-level Transformer in capturing intricate panel details.

Figure: Figure 11. Effect of different shape losses. Compared to the prior method (Korosteleva and Lee, 2022) (w/o $\mathcal{L}_{\text{shape}}$), the proposed shape loss (w/ $\mathcal{L}_{\text{shape}}$) leads to better shape predictions closer to the ground truth. The top and bottom rows show the front and back views, respectively.
Refer to caption: /html/2311.04218/assets/x11.png

##### Effectiveness of the loss functions.

In Section [4.2.1](#S4.SS2.SSS1), we introduce a new panel shape loss $\mathcal{L}_{\text{shape}}$ which better encourages shape similarity between the prediction and ground truth compared to the prior method $\mathcal{L}_{\text{shape-NT}}$ from (Korosteleva and Lee, 2022).
As shown in Table [3](#S5.T3), the proposed shape loss achieves noticeably smaller errors in panel shape (Panel L2).
This improvement is further supported by the visual example in Fig. [11](#S5.F11), which demonstrates that our proposed shape loss is able to recover panel shapes closer to the ground truth.
Remarkably, $\mathcal{L}_{\text{shape}}$ also improves the prediction accuracy of the panel rotation and translation in Table [3](#S5.T3).
We hypothesize that this can be attributed to that our shape loss encourages the model to learn more discriminative panel features, which consequently facilitates more accurate estimations of panel rotation and translation.

As a garment is closely related to the human body wearing it,
we propose an SMPL-based regularization loss $\mathcal{L}_{\text{SMPL}}$ to exploit the human body information for improving garment panel reconstruction.
As shown in Table [3](#S5.T3),
the absence of this loss leads to inaccurate panel rotation and translation predictions, as well as a significant drop in the #Panel accuracy, showing the important role of human body information in understanding the spatial relationship of different panels.
Additionally, Fig. [12](#S5.F12) demonstrates that the model trained without the SMPL loss produces inferior results in terms of panel shape and global 3D parameters. Notably, our proposed $\mathcal{L}_{\text{SMPL}}$ produces plausible results even for occluded regions, emphasizing the significance of incorporating human body information for garment panel reconstruction.

Figure: Figure 12. Effect of the SMPL Loss. The result without $\mathcal{L}_{\text{SMPL}}$ suffers from incorrect spatial arrangement of the panels and noticeable shape errors. In comparison, incorporating $\mathcal{L}_{\text{SMPL}}$ leads to improved performance, generating plausible results even for occluded regions.
Refer to caption: /html/2311.04218/assets/x12.png

Figure: Figure 13. Garment reproduction and editing. Each row shows an example. The first column is the input RGB image, and the second column is the corresponding ground truth sewing pattern. The third column is the sewing pattern recovered by our method. For ease of comparison, we show the back-side panels in the front for the first and fourth examples, where the human is facing backward. The fourth and fifth columns show the recovered garment in T-pose. The sixth column is the reproduction of the input garment, and the last column is a random editing result. The garment textures are manually added.
Refer to caption: /html/2311.04218/assets/x13.png

Figure: Figure 14. Comparison with single-view garment mesh reconstruction methods. Compared to ECON (Xiu et al., 2023) and SMPLicit (Corona et al., 2021), the proposed method achieves high fidelity and realistic details even in occluded areas.
Refer to caption: /html/2311.04218/assets/x14.png

### 5.4. Garment Reproduction and Editing

An important application of our work is the reproduction and editing of garments from a single image.
Specifically, given an input image, we first use the proposed Sewformer to recover its sewing pattern.
As shown in Fig. [13](#S5.F13), our predicted panel shapes exhibit faithful details, such as the depth and angle of the neckline, the width of the hems and sleeves, and the symmetry within and between the panels in the sewing pattern.
The high-quality sewing pattern reconstruction allows us to use physical simulators like Qualoth (Choi and Ko, 2002) to obtain an accurate reproduction of the 3D garment by simulating the sewing pattern on the input human shape and pose.
We estimate the 3D human shape and pose with RSC-Net (Xu et al., 2020).
This process enables flexible editing of the 3D garment as illustrated in Fig. [13](#S5.F13), including modifications to garment textures, human poses, human shapes, and more. These capabilities are highly valuable in procedural production processes.

Figure: Figure 15. Results on real human images. The second, third, and fourth columns are results recovered by the proposed network trained on three different datasets. The proposed algorithm shows strong generalization capabilities, reconstructing garments that closely resemble the input image. Meanwhile, the proposed human texture synthesis network is crucial for reducing domain gap and achieving good generalization ability.
Refer to caption: /html/2311.04218/assets/x15.png

### 5.5. Single-View Garment Mesh Reconstruction

In Fig. [14](#S5.F14), we present a comparison with two state-of-the-art methods, ECON (Xiu et al., 2023) and SMPLicit (Corona et al., 2021), for garment mesh reconstruction from a single image.
As shown in Fig. [14](#S5.F14), while ECON well handles front views, it is susceptible to occlusions and produces unrealistic details and over-smoothed areas for the back views.
SMPLicit addresses this issue by explicitly considering clothing parameters, resulting in plausible outputs for both front and back views. However, due to the oversimplified parameter space, its results are less accurate.
In contrast, our proposed method achieves improved results, demonstrating high fidelity and realistic details even in occluded regions.

### 5.6. Generalization to Real Photos

We present visual results on real human images in Fig. [15](#S5.F15). The proposed algorithm produces garments that closely resemble the input image, demonstrating the strong generalizability of our method.

##### Effectiveness of human texture synthesis.

In Section [3.2](#S3.SS2), we propose a human texture synthesis network to generate realistic training data for our algorithm.
To investigate the effectiveness of the synthesized human texture, we compare the pattern reconstruction results of models trained with different human textures on real input images.
Specifically, we compare three different types of data: 1) training images without human textures; 2) training images rendered with scanned skin textures from the SURREAL dataset (Varol et al., 2017); and 3) images rendered with our textures from the human texture synthesis network.
As “w/o texture” and SURREAL suffer from domain gaps with real data (Fig. [4](#S3.F4)), they result in degraded results as shown in Fig. [15](#S5.F15).
In contrast, our human texture synthesis network produces diverse high-quality human textures, effectively reducing the domain gap and improving the performance on real human photos.

**Table 4. User study of different human textures for sewing pattern reconstruction on real images. Participants are asked to rate the three methods on a scale of 3 (excellent) to 1 (poor) or 0 (fail in simulation).**
| Texture | w/o texture | SURREAL | Ours |
| --- | --- | --- | --- |
| Score $\uparrow$ | 0.822 | 1.12 | 2.35 |

Furthermore, we conduct a user study for a more comprehensive evaluation.
This study uses 51 images randomly selected from the DeepFashion dataset (Liu et al., 2016).
45 subjects are asked to rank the recovered sewing patterns by the models trained using different human textures, with the input image as the reference (1 for poor and 3 for excellent).
We use the averaged scores for measuring the results.
As shown in Table [4](#S5.T4), the model trained with the proposed human textures is clearly preferred over other models, suggesting the effectiveness of our algorithm in handling real-world human photos.

Figure: Figure 16. Generalization to novel patterns. With the template-free network design and the high-quality dataset, the proposed algorithm is able to handle garment topology unseen during training.
Refer to caption: /html/2311.04218/assets/x16.png

##### Generalization to unseen topology.

As demonstrated in Fig.[16](#S5.F16), the proposed algorithm is able to generalize to unseen garment topologies, where the sewing pattern (a one-piece jumpsuit) has not been encountered during training.
This capability can be attributed to two key factors. First, our network is designed without assuming the input garment type.
This stands in contrast to approaches such as (Chen et al., 2022a), which is constrained by predefined panel groups, or (Bhatnagar et al., 2019), which relies on fixed garment templates.
By avoiding these constraints, our network can effectively handle diverse and novel garment topologies.
Second, the SewFactory dataset, specifically designed to prioritize scale and diversity, plays a crucial role in enabling such generalization.
This comprehensive dataset allows the network to learn the fundamental concept of assembling panels into garments, rather than being restricted to the specific styles present in the training data.
As a result, our algorithm exhibits the ability to adapt and generate accurate sewing patterns for unseen garment styles.

Figure: Figure 17. Failure case. The proposed method cannot well handle the classical Chinese dress Qipao whose stitching relations are beyond the settings of our training data. It also lacks the capability to predict unseen accessories such as hats.
Refer to caption: /html/2311.04218/assets/x17.png

##### Limitations.

While exhibiting strong generalization capabilities to real photos, our method encounters challenges when presented with special inputs that deviate significantly from garments in training data.
As illustrated in Fig. [17](#S5.F17) where we show a classical Chinese dress (known as Qipao), the method may exhibit notable errors in reconstructing stitching relations that are not in training data and lack the capability to predict unseen accessories such as hats.
To encompass a broader range of clothing styles and accessories can potentially alleviate this issue and improve the model to handle more diverse fashion elements.

## 6. Discussion and Future Work

In this work, we tackle the challenging task of recovering garment sewing patterns from a single unconstrained human image. Our contributions include the introduction of the SewFactory dataset, which provides a substantial amount of image-and-sewing-pattern pairs to train data-hungry deep learning models. Additionally, we propose a simple yet powerful Transformer model that achieves high-quality results for sewing pattern reconstruction from a single image.

This work has paved the way for accessible, low-cost, and efficient 3D garment design and manipulation.
However, there are still several directions for future exploration and improvement.

First, while the proposed Sewformer demonstrates remarkable performance, it is designed in a minimalistic style, and there is still room for enhancing its capabilities. Further investigation could focus on incorporating more advanced attention mechanisms (Liu et al., 2021) to capture finer details in the sewing patterns, and/or leveraging temporal information from image sequences to enhance the reconstruction.

Second, expanding the SewFactory dataset and incorporating more diverse garment styles, body shapes and poses, and higher variations in rendering conditions would contribute to better generalization and robustness of the model.
Furthermore, considering the interaction between garments and human bodies is an intriguing direction for future work. While the proposed SMPL regularization loss shows an effective exploration in this direction, it is possible to take one step further to employ the human body information in a more explicit way, e.g., directly modeling the connection between body parts and panels.

Lastly, applying the proposed model and dataset in virtual and augmented reality, such as personalized virtual try-on systems, virtual fashion design platforms, or online shopping, could extend the impact of this work into various domains.

## References

- (1)
- Albahar et al. (2021)
Badour Albahar, Jingwan
Lu, Jimei Yang, Zhixin Shu,
Eli Shechtman, and Jia-Bin Huang.
2021.
Pose with Style: Detail-preserving pose-guided
image synthesis with conditional stylegan.
*ACM Transactions on Graphics (TOG)*
40, 6 (2021),
1–11.
- Alldieck et al. (2019)
Thiemo Alldieck, Marcus
Magnor, Bharat Lal Bhatnagar, Christian
Theobalt, and Gerard Pons-Moll.
2019.
Learning to reconstruct people in clothing from a
single RGB camera. In *Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition*.
1175–1186.
- Autodesk, INC. (2019)
Autodesk, INC.
2019.
*Maya*.
[https:/autodesk.com/maya](https:/autodesk.com/maya)
- Bang et al. (2021)
Seungbae Bang, Maria
Korosteleva, and Sung-Hee Lee.
2021.
Estimating Garment Patterns from Static Scan Data.
*Computer Graphics Forum (Eurographics)*
40, 6 (2021),
273–287.
- Bertiche et al. (2020)
Hugo Bertiche, Meysam
Madadi, and Sergio Escalera.
2020.
CLOTH3D: clothed 3d humans. In
*European Conference on Computer Vision*. Springer,
344–359.
- Bhatnagar et al. (2019)
Bharat Lal Bhatnagar,
Garvita Tiwari, Christian Theobalt, and
Gerard Pons-Moll. 2019.
Multi-garment net: Learning to dress 3d people from
images. In *proceedings of the IEEE/CVF
international conference on computer vision*. 5420–5430.
- Chaudhuri et al. (2021)
Bindita Chaudhuri,
Nikolaos Sarafianos, Linda Shapiro, and
Tony Tung. 2021.
Semi-supervised synthesis of high-resolution
editable textures for 3d humans. In *Proceedings of
the IEEE/CVF Conference on Computer Vision and Pattern Recognition*.
7991–8000.
- Chen et al. (2022a)
Xipeng Chen, Guangrun
Wang, Dizhong Zhu, Xiaodan Liang,
Philip HS Torr, and Liang Lin.
2022a.
Structure-Preserving 3D Garment Modeling with
Neural Sewing Machines.
*arXiv preprint arXiv:2211.06701*
(2022).
- Chen et al. (2015)
Xiaowu Chen, Bin Zhou,
Fei-Xiang Lu, Lin Wang,
Lang Bi, and Ping Tan.
2015.
Garment modeling with a depth camera.
*ACM Transactions on Graphics*
34, 6 (2015),
203–1.
- Chen et al. (2022b)
Zhiqin Chen, Kangxue Yin,
and Sanja Fidler. 2022b.
AUV-Net: Learning Aligned UV Maps for Texture
Transfer and Synthesis. In *Proceedings of the
IEEE/CVF Conference on Computer Vision and Pattern Recognition*.
1465–1474.
- Choi and Ko (2002)
Kwang-Jin Choi and
Hyeong-Seok Ko. 2002.
Stable but Responsive Cloth. In
*Proceedings of the 29th Annual Conference on
Computer Graphics and Interactive Techniques*.
Association for Computing Machinery,
New York, NY, USA, 604–611.
[https://doi.org/10.1145/566570.566624](https://doi.org/10.1145/566570.566624)
- Corona et al. (2021)
Enric Corona, Albert
Pumarola, Guillem Alenya, Gerard
Pons-Moll, and Francesc Moreno-Noguer.
2021.
Smplicit: Topology-aware generative model for
clothed people. In *Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition*.
11875–11885.
- Fu et al. (2022)
Jianglin Fu, Shikai Li,
Yuming Jiang, Kwan-Yee Lin,
Chen Qian, Chen Change Loy,
Wayne Wu, and Ziwei Liu.
2022.
Stylegan-human: A data-centric odyssey of human
generation. In *European Conference on Computer
Vision*. Springer, 1–19.
- Georgiev et al. (2018)
Iliyan Georgiev, Thiago
Ize, Mike Farnsworth, Ramón
Montoya-Vozmediano, Alan King, Brecht Van
Lommel, Angel Jimenez, Oscar Anson,
Shinji Ogaki, Eric Johnston,
et al. 2018.
Arnold: A brute-force production path tracer.
*ACM Transactions on Graphics (TOG)*
37, 3 (2018),
1–12.
- Goodfellow et al. (2020)
Ian Goodfellow, Jean
Pouget-Abadie, Mehdi Mirza, Bing Xu,
David Warde-Farley, Sherjil Ozair,
Aaron Courville, and Yoshua Bengio.
2020.
Generative adversarial networks.
*Commun. ACM* 63,
11 (2020), 139–144.
- Goto and Umetani (2021)
Chihiro Goto and
Nobuyuki Umetani. 2021.
Data-driven Garment Pattern Estimation from 3D
Geometries. In *Eurographics*.
The Eurographics Association.
- Grigorev et al. (2021)
Artur Grigorev, Karim
Iskakov, Anastasia Ianina, Renat
Bashirov, Ilya Zakharkin, Alexander
Vakhitov, and Victor Lempitsky.
2021.
Stylepeople: A generative model of fullbody human
avatars. In *Proceedings of the IEEE/CVF Conference
on Computer Vision and Pattern Recognition*. 5151–5160.
- Güler et al. (2018)
Rıza Alp Güler,
Natalia Neverova, and Iasonas
Kokkinos. 2018.
Densepose: Dense human pose estimation in the
wild. In *Proceedings of the IEEE conference on
computer vision and pattern recognition*. 7297–7306.
- Han et al. (2019)
Xintong Han, Xiaojun Hu,
Weilin Huang, and Matthew R Scott.
2019.
Clothflow: A flow-based model for clothed person
generation. In *Proceedings of the IEEE/CVF
international conference on computer vision*. 10471–10480.
- Hasler et al. (2007)
Nils Hasler, Bodo
Rosenhahn, and Hans-Peter Seidel.
2007.
Reverse engineering garments. In
*International Conference on Computer
Vision/Computer Graphics Collaboration Techniques and Applications*.
Springer, 200–211.
- He et al. (2016)
Kaiming He, Xiangyu
Zhang, Shaoqing Ren, and Jian Sun.
2016.
Deep residual learning for image recognition. In
*Proceedings of the IEEE conference on computer
vision and pattern recognition*. 770–778.
- Hewitt et al. (2023)
Charlie Hewitt, Tadas
Baltrušaitis, Erroll Wood, Lohit
Petikam, Louis Florentin, and
Hanz Cuevas Velasquez. 2023.
Procedural Humans for Computer Vision.
*arXiv preprint arXiv:2301.01161*
(2023).
- Huang et al. (2020)
Zeng Huang, Yuanlu Xu,
Christoph Lassner, Hao Li, and
Tony Tung. 2020.
Arch: Animatable reconstruction of clothed humans.
In *Proceedings of the IEEE/CVF Conference on
Computer Vision and Pattern Recognition*. 3093–3102.
- Isola et al. (2017)
Phillip Isola, Jun-Yan
Zhu, Tinghui Zhou, and Alexei A
Efros. 2017.
Image-to-image translation with conditional
adversarial networks. In *Proceedings of the IEEE
conference on computer vision and pattern recognition*.
1125–1134.
- Jeong et al. (2015)
Moon-Hwan Jeong, Dong-Hoon
Han, and Hyeong-Seok Ko.
2015.
Garment capture from a photograph.
*Computer Animation and Virtual Worlds*
26, 3-4 (2015),
291–300.
- Jiang et al. (2020)
Boyi Jiang, Juyong Zhang,
Yang Hong, Jinhao Luo,
Ligang Liu, and Hujun Bao.
2020.
Bcnet: Learning body and cloth shape from a single
image. In *European Conference on Computer
Vision*. Springer, 18–35.
- Johnson et al. (2016)
Justin Johnson, Alexandre
Alahi, and Li Fei-Fei. 2016.
Perceptual losses for real-time style transfer and
super-resolution. In *European conference on
computer vision*. Springer, 694–711.
- Karras et al. (2019)
Tero Karras, Samuli
Laine, and Timo Aila. 2019.
A style-based generator architecture for generative
adversarial networks. In *Proceedings of the
IEEE/CVF conference on computer vision and pattern recognition*.
4401–4410.
- Kennedy and Eberhart (1995)
James Kennedy and
Russell Eberhart. 1995.
Particle swarm optimization. In
*Proceedings of ICNN’95-international conference on
neural networks*, Vol. 4. IEEE,
1942–1948.
- Korosteleva and Lee (2021)
Maria Korosteleva and
Sung-Hee Lee. 2021.
Generating Datasets of 3D Garments with Sewing
Patterns. In *Proceedings of the Neural Information
Processing Systems Track on Datasets and Benchmarks*,
Vol. 1.
- Korosteleva and Lee (2022)
Maria Korosteleva and
Sung-Hee Lee. 2022.
NeuralTailor: Reconstructing Sewing Pattern
Structures from 3D Point Clouds of Garments.
*ACM Transactions on Graphics (SIGGRAPH)*
41, 4 (2022),
16 pages.
- Lassner et al. (2017)
Christoph Lassner, Gerard
Pons-Moll, and Peter V Gehler.
2017.
A generative model of people in clothing. In
*Proceedings of the IEEE International Conference on
Computer Vision*. 853–862.
- Lazova et al. (2019)
Verica Lazova, Eldar
Insafutdinov, and Gerard Pons-Moll.
2019.
360-degree textures of people in clothing from a
single image. In *2019 International Conference on
3D Vision (3DV)*. IEEE, 643–653.
- Liu et al. (2018)
Kaixuan Liu, Xianyi Zeng,
Pascal Bruniaux, Xuyuan Tao,
Xiaofeng Yao, Victoria Li, and
Jianping Wang. 2018.
3D interactive garment pattern-making technology.
*Computer-Aided Design*
104 (2018), 113–124.
- Liu et al. (2021)
Ze Liu, Yutong Lin,
Yue Cao, Han Hu, Yixuan
Wei, Zheng Zhang, Stephen Lin, and
Baining Guo. 2021.
Swin transformer: Hierarchical vision transformer
using shifted windows. In *Proceedings of the
IEEE/CVF international conference on computer vision*.
10012–10022.
- Liu et al. (2016)
Ziwei Liu, Ping Luo,
Shi Qiu, Xiaogang Wang, and
Xiaoou Tang. 2016.
Deepfashion: Powering robust clothes recognition
and retrieval with rich annotations. In
*Proceedings of the IEEE conference on computer
vision and pattern recognition*. 1096–1104.
- Loper et al. (2015)
Matthew Loper, Naureen
Mahmood, Javier Romero, Gerard
Pons-Moll, and Michael J Black.
2015.
SMPL: A skinned multi-person linear model.
*ACM transactions on graphics (TOG)*
34, 6 (2015),
1–16.
- Ma et al. (2020)
Qianli Ma, Jinlong Yang,
Anurag Ranjan, Sergi Pujades,
Gerard Pons-Moll, Siyu Tang, and
Michael J Black. 2020.
Learning to dress 3d people in generative
clothing. In *Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition*.
6469–6478.
- Mahmood et al. (2019)
Naureen Mahmood, Nima
Ghorbani, Nikolaus F Troje, Gerard
Pons-Moll, and Michael J Black.
2019.
AMASS: Archive of motion capture as surface
shapes. In *Proceedings of the IEEE/CVF
international conference on computer vision*. 5442–5451.
- Meng et al. (2021)
Chenlin Meng, Yutong He,
Yang Song, Jiaming Song,
Jiajun Wu, Jun-Yan Zhu, and
Stefano Ermon. 2021.
Sdedit: Guided image synthesis and editing with
stochastic differential equations. In
*International Conference on Learning
Representations*.
- Moon et al. (2022)
Gyeongsik Moon, Hyeongjin
Nam, Takaaki Shiratori, and Kyoung Mu
Lee. 2022.
3D clothed human reconstruction in the wild. In
*Computer Vision–ECCV 2022: 17th European
Conference, Tel Aviv, Israel, October 23–27, 2022, Proceedings, Part II*.
Springer, 184–200.
- Narain et al. (2012)
Rahul Narain, Armin
Samii, and James F O’brien.
2012.
Adaptive anisotropic remeshing for cloth
simulation.
*ACM transactions on graphics (TOG)*
31, 6 (2012),
1–10.
- Patel et al. (2020)
Chaitanya Patel,
Zhouyingcheng Liao, and Gerard
Pons-Moll. 2020.
Tailornet: Predicting clothing in 3d as a function
of human pose, shape and garment style. In
*Proceedings of the IEEE/CVF Conference on Computer
Vision and Pattern Recognition*. 7365–7375.
- Pumarola et al. (2019)
Albert Pumarola, Jordi
Sanchez-Riera, Gary Choi, Alberto
Sanfeliu, and Francesc Moreno-Noguer.
2019.
3dpeople: Modeling the geometry of dressed humans.
In *Proceedings of the IEEE/CVF international
conference on computer vision*. 2242–2251.
- Rombach et al. (2022)
Robin Rombach, Andreas
Blattmann, Dominik Lorenz, Patrick
Esser, and Björn Ommer.
2022.
High-resolution image synthesis with latent
diffusion models. In *Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition*.
10684–10695.
- Ronneberger et al. (2015)
Olaf Ronneberger, Philipp
Fischer, and Thomas Brox.
2015.
U-net: Convolutional networks for biomedical image
segmentation. In *International Conference on
Medical image computing and computer-assisted intervention*. Springer,
234–241.
- Saito et al. (2019)
Shunsuke Saito, Zeng
Huang, Ryota Natsume, Shigeo Morishima,
Angjoo Kanazawa, and Hao Li.
2019.
Pifu: Pixel-aligned implicit function for
high-resolution clothed human digitization. In
*Proceedings of the IEEE/CVF international
conference on computer vision*. 2304–2314.
- Saito et al. (2020)
Shunsuke Saito, Tomas
Simon, Jason Saragih, and Hanbyul
Joo. 2020.
Pifuhd: Multi-level pixel-aligned implicit function
for high-resolution 3d human digitization. In
*Proceedings of the IEEE/CVF Conference on Computer
Vision and Pattern Recognition*. 84–93.
- Sarkar et al. (2021a)
Kripasindhu Sarkar,
Vladislav Golyanik, Lingjie Liu, and
Christian Theobalt. 2021a.
Style and pose control for image synthesis of
humans from a single monocular view.
*arXiv preprint arXiv:2102.11263*
(2021).
- Sarkar et al. (2021b)
Kripasindhu Sarkar,
Lingjie Liu, Vladislav Golyanik, and
Christian Theobalt. 2021b.
Humangan: A generative model of human images. In
*2021 International Conference on 3D Vision (3DV)*.
IEEE, 258–267.
- Sarkar et al. (2020)
Kripasindhu Sarkar,
Dushyant Mehta, Weipeng Xu,
Vladislav Golyanik, and Christian
Theobalt. 2020.
Neural re-rendering of humans from a single image.
In *European Conference on Computer Vision*.
Springer, 596–613.
- Schmidt et al. (2006)
Ryan Schmidt, Cindy
Grimm, and Brian Wyvill.
2006.
Interactive decal compositing with discrete
exponential maps.
In *ACM SIGGRAPH 2006 Papers*.
605–613.
- Sharp and Crane (2018)
Nicholas Sharp and
Keenan Crane. 2018.
Variational surface cutting.
*ACM Transactions on Graphics (TOG)*
37, 4 (2018),
1–13.
- Tiwari et al. (2020)
Garvita Tiwari, Bharat Lal
Bhatnagar, Tony Tung, and Gerard
Pons-Moll. 2020.
Sizer: A dataset and model for parsing 3d clothing
and learning size sensitive 3d clothing. In
*European Conference on Computer Vision*. Springer,
1–18.
- Tiwari and Bhowmick (2021)
Lokender Tiwari and
Brojeshwar Bhowmick. 2021.
DeepDraper: Fast and Accurate 3D Garment Draping
over a 3D Human Body. In *Proceedings of the
IEEE/CVF International Conference on Computer Vision*.
1416–1426.
- Varol et al. (2017)
Gül Varol, Javier
Romero, Xavier Martin, Naureen Mahmood,
Michael J. Black, Ivan Laptev, and
Cordelia Schmid. 2017.
Learning from Synthetic Humans. In
*CVPR*.
- Vaswani et al. (2017)
Ashish Vaswani, Noam
Shazeer, Niki Parmar, Jakob Uszkoreit,
Llion Jones, Aidan N Gomez,
Łukasz Kaiser, and Illia
Polosukhin. 2017.
Attention is all you need.
*Advances in neural information processing
systems* 30 (2017).
- Wang et al. (2019)
Miao Wang, Guo-Ye Yang,
Ruilong Li, Run-Ze Liang,
Song-Hai Zhang, Peter M Hall, and
Shi-Min Hu. 2019.
Example-guided style-consistent image synthesis
from semantic labeling. In *Proceedings of the
IEEE/CVF Conference on Computer Vision and Pattern Recognition*.
1495–1504.
- Wang et al. (2018b)
Nanyang Wang, Yinda
Zhang, Zhuwen Li, Yanwei Fu,
Wei Liu, and Yu-Gang Jiang.
2018b.
Pixel2mesh: Generating 3d mesh models from single
rgb images. In *Proceedings of the European
conference on computer vision (ECCV)*. 52–67.
- Wang et al. (2018a)
Tuanfeng Y. Wang, Duygu
Ceylan, Jovan Popovic, and Niloy J.
Mitra. 2018a.
Learning a Shared Shape Space for Multimodal
Garment Design.
*ACM Trans. Graph.* 37,
6 (2018), 1:1–1:14.
[https://doi.org/10.1145/3272127.3275074](https://doi.org/10.1145/3272127.3275074)
- Weng et al. (2020)
Shuchen Weng, Wenbo Li,
Dawei Li, Hongxia Jin, and
Boxin Shi. 2020.
Misc: Multi-condition injection and
spatially-adaptive compositing for conditional person image synthesis. In
*Proceedings of the IEEE/CVF Conference on Computer
Vision and Pattern Recognition*. 7741–7749.
- Xian et al. (2018)
Wenqi Xian, Patsorn
Sangkloy, Varun Agrawal, Amit Raj,
Jingwan Lu, Chen Fang,
Fisher Yu, and James Hays.
2018.
Texturegan: Controlling deep image synthesis with
texture patches. In *Proceedings of the IEEE
conference on computer vision and pattern recognition*.
8456–8465.
- Xiang et al. (2022)
Donglai Xiang, Timur
Bagautdinov, Tuur Stuyck, Fabian Prada,
Javier Romero, Weipeng Xu,
Shunsuke Saito, Jingfan Guo,
Breannan Smith, Takaaki Shiratori,
et al. 2022.
Dressing avatars: Deep photorealistic appearance
for physically simulated clothing.
*ACM Transactions on Graphics (TOG)*
41, 6 (2022),
1–15.
- Xiu et al. (2023)
Yuliang Xiu, Jinlong
Yang, Xu Cao, Dimitrios Tzionas, and
Michael J. Black. 2023.
ECON: Explicit Clothed humans Optimized via Normal
integration. In *Proceedings of the IEEE/CVF
Conference on Computer Vision and Pattern Recognition (CVPR)*.
- Xu et al. (2020)
Xiangyu Xu, Hao Chen,
Francesc Moreno-Noguer, Laszlo A Jeni,
and Fernando De la Torre.
2020.
3D Human Shape and Pose from a Single
Low-Resolution Image with Self-Supervised Learning. In
*ECCV*.
- Xu and Loy (2021)
Xiangyu Xu and
Chen Change Loy. 2021.
3D human texture estimation from a single image
with transformers. In *Proceedings of the IEEE/CVF
International Conference on Computer Vision*. 13849–13858.
- Xu et al. (2017)
Xiangyu Xu, Deqing Sun,
Jinshan Pan, Yujin Zhang,
Hanspeter Pfister, and Ming-Hsuan
Yang. 2017.
Learning to super-resolve blurry face and text
images. In *Proceedings of the IEEE international
conference on computer vision*. 251–260.
- Yang et al. (2018)
Shan Yang, Zherong Pan,
Tanya Amert, Ke Wang,
Licheng Yu, Tamara Berg, and
Ming C Lin. 2018.
Physics-inspired garment recovery from a
single-view image.
*ACM Transactions on Graphics*
37, 5 (2018),
1–14.
- Yang et al. (2022)
Zhuoqian Yang, Shikai Li,
Wayne Wu, and Bo Dai.
2022.
3DHumanGAN: Towards Photo-Realistic 3D-Aware Human
Image Generation.
*arXiv preprint arXiv:2212.07378*
(2022).
- Zhang et al. (2017)
Chao Zhang, Sergi
Pujades, Michael J Black, and Gerard
Pons-Moll. 2017.
Detailed, accurate, human shape estimation from
clothed 3D scan sequences. In *Proceedings of the
IEEE Conference on Computer Vision and Pattern Recognition*.
4191–4200.
- Zhang et al. (2020)
Pan Zhang, Bo Zhang,
Dong Chen, Lu Yuan, and
Fang Wen. 2020.
Cross-domain correspondence learning for
exemplar-based image translation. In *Proceedings
of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*.
5143–5153.
- Zhao et al. (2020)
Fang Zhao, Shengcai Liao,
Kaihao Zhang, and Ling Shao.
2020.
Human parsing based texture transfer from single
image to 3D human via cross-view consistency.
*Advances in Neural Information Processing
Systems* 33 (2020),
14326–14337.
- Zhao et al. (2021)
Fang Zhao, Wenhao Wang,
Shengcai Liao, and Ling Shao.
2021.
Learning anchored unsigned distance functions with
gradient direction alignment for single-view garment reconstruction. In
*Proceedings of the IEEE/CVF International
Conference on Computer Vision*. 12674–12683.
- Zhu et al. (2020)
Heming Zhu, Yu Cao,
Hang Jin, Weikai Chen,
Dong Du, Zhangye Wang,
Shuguang Cui, and Xiaoguang Han.
2020.
Deep Fashion3D: A dataset and benchmark for 3D
garment reconstruction from single images. In
*European Conference on Computer Vision*.