<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related Work
  - Tight-Fitting Clothing.
  - Loose-Fitting Clothing.
  - Diffusion Model.
- 3. Garment Representation Model
  - 3.1. Implicit Sewing Patterns
    - Formalization.
    - Training.
  - 3.2. Extending ISP with a Diffusion Model
- 4. Reconstructing Garments from Monocular Images
  - 4.1. Observations from Images
  - 4.2. Mapping from Pixel Space to UV Space and 3D Space
    - To 3D Space.
    - To UV Space.
    - Training.
  - 4.3. Fitting
    - 4.3.1. Recovering the Rest Geometry.
    - 4.3.2. Recovering Deformed Geometry.
    - 4.3.3. Garment Refinement.
    - 4.3.4. Refining Body
- 5. Experiments
  - 5.1. Dataset and Evaluation Metrics
  - 5.2. Results
  - 5.3. Ablation Study
  - 5.4. Downstream Applications
    - Retargeting
    - Texture Editing
- 6. Conclusion
  - Limitations
- Acknowledgments
- References

## Abstract

Abstract. Reconstructing 3D clothed humans from images is fundamental to applications like virtual try-on, avatar creation, and mixed reality. While recent advances have enhanced human body recovery, accurate reconstruction of garment geometry—especially for loose-fitting clothing—remains an open challenge.
We present a novel method for high-fidelity 3D garment reconstruction from single images that bridges 2D and 3D representations. Our approach combines Implicit Sewing Patterns (ISP) with a generative diffusion model to learn rich garment shape priors in a 2D UV space. A key innovation is our mapping model that establishes correspondences between 2D image pixels, UV pattern coordinates, and 3D geometry, enabling joint optimization of both 3D garment meshes and the corresponding 2D patterns by aligning learned priors with image observations.
Despite training exclusively on synthetically simulated cloth data, our method generalizes effectively to real-world images, outperforming existing approaches on both tight- and loose-fitting garments. The reconstructed garments maintain physical plausibility while capturing fine geometric details, enabling downstream applications including garment retargeting and texture manipulation.

## 1. Introduction

Recovering the pose and shape of people’s bodies, along with the shape of their garments, solely from images has many applications. They include fashion design, virtual try-on, creating 3D avatars, telepresence, and immersive VR/AR.
Recent years have seen tremendous progress in modeling people wearing tight-fitting clothing both in terms of body poses (Bogo et al., 2016; Lassner et al., 2017; Kanazawa et al., 2018; Kolotouros et al., 2019; Pavlakos et al., 2019; Georgakis et al., 2020; Moon and Lee, 2020; Li et al., 2021a, b; Xu et al., 2024a; Li et al., 2024c) and 3D shape of the clothes (Danerek et al., 2017; Bhatnagar et al., 2019; Jiang et al., 2020; Corona et al., 2021; Moon et al., 2022; Li et al., 2022; DeLuigi et al., 2023; Li et al., 2023). However, accurately modeling clothing, especially when it fits loosely on the body, remains a challenge. Most current work relies on a single 3D model to jointly represent the body and its clothing. While this can produce visually impressive reconstruction results, this fused representation of humans and their garments makes it impossible to perform realistic cloth simulation or virtual try-on.

Figure: Figure 2. Pipeline. Given an image of a clothed person, we first estimate the front normal $\mathbf{N}_{F}$ of the target garment, and the SMPL body model which is used to render the body part segmentation ($\mathbf{S}_{F}$, $\mathbf{S}_{B}$) and depth ($\mathbf{D}_{F}^{b}$, $\mathbf{D}_{B}^{b}$) images. The back normal $\mathbf{N}_{B}$ of the garment is estimated subsequently by the diffusion model $\boldsymbol{\epsilon}_{\theta}^{n}$. We then predict the UV-coordinate ($\mathbf{C}_{F}$, $\mathbf{C}_{B}$) and the depth ($\mathbf{D}_{F}^{g}$, $\mathbf{D}_{B}^{g}$) images from the garment normal and body estimations with the mapping model $\boldsymbol{\epsilon}_{\theta}^{m}$. The incomplete UV positional map $\tilde{\mathcal{U}}$ is produced from them using the camera backprojection. Finally, we fit $\tilde{\mathcal{U}}$ to DISP to recover the complete UV positional map $\hat{\mathcal{U}}$ and the corresponding garment mesh $\mathbf{G}$, which is further improved by the refinement.
Refer to caption: x2.png

Thus, independent body and garment models are needed. Such modeling is made difficult by the intricate structure of clothing. Since garments are thin surfaces with near-infinite degrees of freedom, they undergo complex deformations caused by dynamic factors. The many design styles and shape variations of clothing items introduce further complexity, making the modeling process even more challenging and the acquisition of real 3D data more difficult. In turn, this impedes the deployment of learning-based methods for garment reconstruction. To address these challenges, several works (Danerek et al., 2017; Bhatnagar et al., 2019; Jiang et al., 2020; Casado-Elvira et al., 2022; Liu et al., 2023a) rely on pre-designed mesh templates to define the garment geometry, and employ linear blend skinning (LBS) (Loper et al., 2015) of the underlying body model to capture deformations caused by body motion. However, this requires mesh templates for the clothes, which limits modeling flexibility and generality. Furthermore, while skinning is effective for tight-fitting clothing, it struggles to accurately model loose-fitting clothing that often moves far from the body. This was addressed in (Li et al., 2024b) by starting from the so-called Implicit Sewing Patterns (ISP) model (Li et al., 2023) that represents garments in terms of a set of individual 2D panels and 3D surfaces associated to these panels, and then applying a deformation model to the 3D surfaces so that they can deviate substantially from the body shape. These deformations are conditioned on normals estimated from an input image of the target garment, which are learned from synthetic mesh data featuring loose clothing.

The approach of (Li et al., 2024b) is effective but tends to over-smooth the results. This is in part because different 3D shapes can give rise to very similar images, making it difficult to properly train a network to predict high-fidelity surface details and complex deformations from a single image. Furthermore, some parts of the garments are systematically occluded in images of people wearing them. To overcome these limitations, we introduce three diffusion schemes:

- (1)
to learn a shape prior that captures complex garment shapes,
- (2)
to complement image information in occluded parts of the garments,
- (3)
to map the 2D image to 3D and UV spaces so as to recover plausible 3D shapes by fitting them to the shape prior.

Fig. [2](https://arxiv.org/html/2504.08353v2#S1.F2) depicts the resulting processing pipeline. We demonstrate that our method can recover realistic 3D models for various garments. It recovers more details and achieves higher reconstruction accuracy than existing approaches. Furthermore, our reconstructed meshes are readily usable by downstream applications, such as garment retargeting and texture editing. Codes are at [Github](https://github.com/liren2515/DMap.git).

## 2. Related Work

##### Tight-Fitting Clothing.

Recent advances in clothed human reconstruction have primarily focused on clothing that adheres relatively closely to the body, thereby significantly limiting the diversity of garment shapes that can be accurately represented. These approaches can be broadly categorized into two main groups.

The first category includes methods that represent the body and garment using a single 3D model. For instance, (Jackson et al., 2018; Zheng et al., 2019) employ voxel-based representations generated by volumetric regression networks to model 3D clothed humans. Other works (Saito et al., 2019, 2020; He et al., 2020; Alldieck et al., 2022) utilize pixel-aligned implicit functions to define 3D occupancy fields or signed distance fields for clothed humans. In (Alldieck et al., 2019a, b), displacement vectors or UV maps are used to capture deviations from the SMPL parametric body model (Loper et al., 2015). Similarly, approaches such as (Huang et al., 2020; He et al., 2021; Xiu et al., 2022; Zheng et al., 2021) combine parametric body models with implicit representations to enhance robustness to changes in body pose. While effective, these methods have significant limitations: because the body and clothing are jointly modeled, they cannot disentangle the garment surface from the body surface. This limitation hinders downstream applications and makes it difficult to model loose garments whose motion can behave independently of the body.

The second category consists of methods that explicitly model garments as separate surfaces that interact with the body. DeepGarment (Danerek et al., 2017), MGN (Bhatnagar et al., 2019), and BCNet (Jiang et al., 2020) employ neural networks trained on synthetic RGB images to predict vertex positions for predefined mesh templates. Other approaches, such as (Casado-Elvira et al., 2022; Liu et al., 2023a), optimize vertex positions based on estimated surface normals to recover fine wrinkle details. However, the reliance on predefined mesh templates inherently limits the range of garment shapes these methods can handle. Additionally, being trained on synthetic RGB data often results in poor reconstructions when applied to real-world images. To address these shortcomings, methods like SMPLicit (Corona et al., 2021), DIG (Li et al., 2022), and ClothWild (Moon et al., 2022) leverage Signed Distance Functions (SDF) to reconstruct a wide variety of garment meshes using segmentation masks derived from RGB images. However, representing non-watertight garment surfaces with an SDF requires enclosing them within watertight surfaces of a minimum thickness, which compromises modeling accuracy and hinders subsequent refinement. To overcome this, approaches such as (Guillard et al., 2022; DeLuigi et al., 2023) adopt Unsigned Distance Functions (UDF). While UDF avoid watertight constraints, they introduce robustness issues: learning a sharp and clean 0-isosurface for UDF is challenging for neural networks, often resulting in inaccuracies that manifest as holes and artifacts in reconstructed models.
The Implicit Sewing Patterns (ISP) model of (Li et al., 2023) effectively addresses the issues of generality, accuracy, and robustness by its 2D pattern and UV parameterization. However, since (Li et al., 2023) is trained specifically for draping, it struggles to capture the large deformations caused by dynamic garment motion.

##### Loose-Fitting Clothing.

Loose-fitting clothing is significantly more challenging to reconstruct due to its large shape variations and free-flowing nature, which keeps it far from the body. Some recent works (Yang et al., 2018; Zhu et al., 2020, 2022) rely on complex physics simulations or feature line estimation to align surface reconstructions with input images. However, their reliance on garment templates limits their generality, similar to the limitations faced by other template-based methods discussed earlier. Point-based approaches, such as those proposed in (Zakharkin et al., 2021; Srivastava et al., 2022; Ma et al., 2022), aim to reconstruct generic clothing. Unfortunately, point clouds are not inherently well-suited for downstream applications like cloth simulation and animation. While (Srivastava et al., 2022) employs a modified Poisson Surface Reconstruction (PSR) technique to generate garment surfaces from point clouds, it often produces results with incorrect geometry. ECON (Xiu et al., 2023), leveraging techniques like normal integration and shape completion, achieves visually appealing reconstructions of individuals wearing loose clothing. However, it still generates a single watertight mesh that tightly binds the body and garment together, which limits its applicability for tasks such as cloth simulation and recreation.

Recently, GaRec (Li et al., 2024b) introduced a method that combines the ISP model (Li et al., 2023) with an image-conditioned deformation model to reconstruct loose-fitting clothing. Because different 3D shapes can result in similar images, it struggles to capture high-fidelity surface details and complex deformations, particularly for unseen or occluded parts that cannot be observed from monocular images. However, our method uses diffusion models to supplement, lift up, and map the 2D image observations to 3D, and fit a deformation prior to it, resulting in high-fidelity 3D reconstruction for the garment.
GarVerseLOD (Luo et al., 2024) addresses garment reconstruction by creating a large-scale hierarchical dataset and training implicit models for garment recovery. However, constructing such a dataset requires significant manual effort from professional artists. In contrast, our method only relies on synthetic data generated by using readily accessible cloth simulation tools (blender, 2018; Designer, 2018), greatly reducing the need for manual intervention.

##### Diffusion Model.

Diffusion models (Ho et al., 2020; Song et al., 2021) are a class of generative models that excel at learning complex data distributions through score matching. These models generate high-quality samples via an iterative denoising process and have demonstrated state-of-the-art performance across a variety of image-based generative tasks (Dhariwal and Nichol, 2021; Chung et al., 2022a, b; Rombach et al., 2022). Beyond 2D tasks, diffusion models have also been applied to various 3D domains, including text-to-3D generation (Xu et al., 2024b; Poole et al., 2022), image-to-3D generation (Müller et al., 2023; Xu et al., 2024b; Anciukevičius et al., 2023; Liu et al., 2023b), and point cloud synthesis (Tyszkiewic et al., 2023; Melas-Kyriazi et al., 2023). Recently, diffusion-based shape priors have been introduced for garment reconstruction (Guo et al., 2024; Li et al., 2024a), which leverage UV maps for garment parameterization. However, these methods require point clouds as 3D measurements of garments and do not account for body-garment interaction during reconstruction. In contrast, our proposed method reconstructs 3D garments directly from monocular 2D images while accurately modeling both the garment and the body.

## 3. Garment Representation Model

In this section, we define our garment representation model, DISP. It defines a reconstruction prior we use when fitting it to images, as discussed in the following section. DISP relies on Implicit Sewing Pattern (ISP) (Li et al., 2023) to model the garment rest geometry. ISP uses UV positional maps to model the geometry of different garments but is limited by only producing a single UV map for a specific garment, which is not enough to represent the many possible answers. To address this issue, we extend ISP into DISP by incorporating a diffusion model to capture the complex garment shapes caused by body motion. It is depicted by the gray network in Fig. [2](https://arxiv.org/html/2504.08353v2#S1.F2). In the remainder of this section, we first describe ISP and then our proposed extension.

### 3.1. Implicit Sewing Patterns

##### Formalization.

Implicit Sewing Patterns (ISP) (Li et al., 2023) is a garment model based on the sewing patterns used in the fashion industry to design and manufacture clothes. A sewing pattern is made of several 2D panels along with stitch information for assembling them together. The 2D panels and the stitching are implicitly modeled using a 2D signed distance field (SDF) and a 2D label field, respectively. For a specific garment, its corresponding latent code $\mathbf{z}$, and a point $\mathbf{u}$ in the 2D UV space $\Omega=[-1,1]^{2}$, the ISP model outputs the signed distance $s$ to the panel boundary and a label $l$ using a fully connected network $\mathcal{I}_{\mathbf{\Theta}}$ as

$$ (1) $(s,l)=\mathcal{I}_{\mathbf{\Theta}}(\mathbf{u},\mathbf{z})\;.$ $$

The zero crossing of the SDF defines the shape of the panel, with $s<0$ indicating that $\mathbf{u}$ is within the panel and $s>0$ indicating that $\mathbf{u}$ is outside the panel. The label $l$ encodes the stitch information, instructing which panel boundaries should be stitched together. To map the 2D sewing patterns to 3D surfaces, a UV parameterization function $\mathcal{A}_{\Phi}$ is learned to perform the 2D-to-3D mapping

$$ (2) $\mathbf{X}=\mathcal{A}_{\Phi}(\mathbf{u},\mathbf{z})\;,$ $$

where $\mathbf{X}\in\mathbb{R}^{3}$ represents the 3D position of $\mathbf{u}$. In essence, ISP registers different garments onto a unified UV space and establishes the mapping functions between points in UV space and the 3D garment surfaces. The shape of SDF’s 0-crossing defines the geometry of garment in its rest state. As ISP is a differentiable representation, we can easily fit a latent code $\mathbf{z}$ to arbitrary masks or contours of the panels to recover the corresponding garment geometry.

##### Training.

Training ISP requires the 2D sewing patterns of rest-state 3D garments. However, they are not available in most garment datasets, e.g. CLOTH3D (Bertiche et al., 2020). Following the garment flattening strategy of (Li et al., 2024b; Pietroni et al., 2022), we cut the garment mesh of CLOTH3D into front and back surfaces according to predefined cutting rules and then flatten them into 2D panels by minimizing an as-rigid-as-possible energy (Liu et al., 2008).
For each garment in the dataset, a front and a back panel are generated as its sewing pattern. By using the paired 2D sewing patterns and their 3D meshes, we learn the weights of the ISP model $(\mathcal{I}_{\mathbf{\Theta}},\mathcal{A}_{\Phi})$ with the training procedure of (Li et al., 2023).

### 3.2. Extending ISP with a Diffusion Model

For a specific garment, the UV parameterization function of ISP only produces a single UV positional map $\mathcal{U}_{r}$ to model its 3D shape in the rest state

$$ (3) $\mathcal{U}_{r}[u,v]=\begin{cases}\mathcal{A}_{\Phi}(\mathbf{u},\mathbf{z}),& \mbox{if }s_{\mathbf{u}}\leq 0\\ \varnothing,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;,$ $$

where $\mathbf{u}=(u,v)$, $s_{\mathbf{u}}$ is the SDF value of $\mathbf{u}$, $[\cdot,\cdot]$ denotes the standard array addressing and $\varnothing=(-1,-1,-1)$ indicates the region outside the panel.
When dressed on the body, the garment can have various deformations due to the motion of the underlying body, which is not able to be modeled by ISP solely. Inspired by (Guo et al., 2024; Li et al., 2024a), we incorporate a diffusion model into ISP to capture these possible deformations by generating plausible UV maps.

Given the deformed garments worn on the body whose rest states are modeled by ISP as Eq. [3](https://arxiv.org/html/2504.08353v2#S3.E3), we write the corresponding UV maps

$$ (4) $\mathcal{U}[u,v]=\begin{cases}\mathbf{V},&\mbox{if }s_{\mathbf{u}}\leq 0\\ \varnothing,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;,$ $$

where $\mathbf{V}\in\mathbb{R}^{3}$ is the corresponding position on the deformed mesh surface for the UV point $\mathbf{u}=(u,v)$. Each $\mathcal{U}$ represents a specific deformed shape for a particular garment. We use a diffusion model (Ho et al., 2020) $\boldsymbol{\epsilon}_{\theta}$ to learn the distribution of plausible deformations represented by $\mathcal{U}$.

For each garment sample, we generate its UV map $\mathcal{U}$ according to Eq. [4](https://arxiv.org/html/2504.08353v2#S3.E4), along with a panel mask $\mathcal{M}$ as

$$ (5) $\mathcal{M}[u,v]=\begin{cases}1,&\mbox{if }s_{\mathbf{u}}\leq 0\\ 0,&\mbox{if }s_{\mathbf{u}}>0\end{cases}\;.$ $$

$\mathcal{M}$ depicts the panel shape, which itself encodes the 3D geometry of the canonical rest garment. We concatenate $\mathcal{U}$ and $\mathcal{M}$ along the channel dimension to form the training samples and train the network $\boldsymbol{\epsilon}_{\theta}$ on them.
After training, the diffusion model and ISP together form the garment model DISP.

## 4. Reconstructing Garments from Monocular Images

Monocular images yield 2D observations for non-occluded regions. We rely on a generative diffusion model to complement image information in occluded parts. By mapping the 2D image information to 3D and UV spaces, we then enable their fitting to DISP for the recovery of realistic 3D garments, even in parts that are not visible.

### 4.1. Observations from Images

Recent advances in image segmentation (Kirillov et al., 2023; Ravi et al., 2024), normal estimation (Bae and Davison, 2024; Khirodkar et al., 2025) and human mesh recovery (Goel et al., 2023; Stathopoulos et al., 2024) can be used to extract accurate observations from an image of a clothed person. In this manner,
we first segment the target garment using (Kirillov et al., 2023; Li et al., 2020) and estimate its normals $\mathbf{N}_{F}$ using (Khirodkar et al., 2025). To model the body underneath, we use SMPL (Loper et al., 2015), which relies on two sets of parameters $(\beta,\theta)$ to describe the body shape and pose respectively. The SMPL parameters are estimated from the image by (Goel et al., 2023) to infer the 3D body shape, which are then used to render front and back body part segmentations $\mathbf{S}_{F}$ and $\mathbf{S}_{B}$, along with front and back depth images $\mathbf{D}_{F}^{b}$ and $\mathbf{D}_{B}^{b}$, as shown in the top left of Fig. [2](https://arxiv.org/html/2504.08353v2#S1.F2).

To estimate the invisible normals, typically in the back, $\mathbf{N}_{B}$ as shown in Fig. [2](https://arxiv.org/html/2504.08353v2#S1.F2), we use the estimated normal $\mathbf{N}_{F}$ to guide a conditional diffusion model $\boldsymbol{\epsilon}_{\theta}^{n}$. The denoising process of $\boldsymbol{\epsilon}_{\theta}^{n}$ is conditioned on the visible normals $\mathbf{N}_{F}$, the front and back segmentation images $\mathbf{S}=(\mathbf{S}_{F},\mathbf{S}_{B})$, the body depth maps $\mathbf{D}=(\mathbf{D}_{F}^{b},\mathbf{D}_{B}^{b})$. It is learned by minimizing the loss

$$ (6) $\mathcal{L}=\mathbb{E}_{t,\mathbf{N}_{B},\boldsymbol{\epsilon}}\|\boldsymbol{ \epsilon}-\boldsymbol{\epsilon}_{\theta}^{n}\left(\sqrt{\bar{\alpha}_{t}} \mathbf{N}_{B}+\sqrt{1-\bar{\alpha}_{t}}\boldsymbol{\epsilon},\mathbf{N}_{F}, \mathbf{S},\mathbf{D},t\right)\|^{2}\;.$ $$

The conditioning images of the garment and body provide information to generate plausible normals for the back of the garment. As will be shown in our experiments, the back normal estimation $\mathbf{N}_{B}$ provides additional constraints to regularize the garment fitting process, which improves the fidelity of the reconstruction.

### 4.2. Mapping from Pixel Space to UV Space and 3D Space

Figure: Figure 3. Mapping between pixel, 3D, and UV spaces. The pixel $(x,y)$ is mapped to $(X,Y,Z)$ in the 3D space using the estimated depth $d$ and the camera backprojection $P^{-1}$, and to $(u,v)$ in the UV space using the estimated UV coordinates $(u,v,\sigma)$. The dash line indicates that $(X,Y,Z)$ and $(u,v)$ are connected indirectly through $(x,y)$.
Refer to caption: x3.png

The normal estimations provide observations in pixel space, while the garment model DISP is learned in the UV space of the garment panels, and the garment surface resides in the 3D space. To reconstruct 3D garments using DISP, it is thus necessary to connect these three different spaces. To this end, we introduce a mapping function that translates image observations from the pixel space to both the UV space and the 3D space, as illustrated by Fig. [3](https://arxiv.org/html/2504.08353v2#S4.F3).

##### To 3D Space.

Since the depth and surface normal are closely related in terms of 3D geometry, we estimate the garment depth image $\mathbf{D}^{g}$ from normal estimations $\mathbf{N}$ conditioned on the body depth $\mathbf{D}^{b}$. For the foreground pixel $(x,y)$, its absolute depth value is $d=\mathbf{D}^{g}[x,y]$. By leveraging the camera projection $P$, we can have the 3D coordinate $(X,Y,Z)$ for each pixel

$$ (7) $(X,Y,Z)=P^{-1}(x,y,d)\;,$ $$

where $P^{-1}$ denotes the camera backprojection. Through Eq. [7](https://arxiv.org/html/2504.08353v2#S4.E7), we establish the mapping from the pixel space to the 3D space.

##### To UV Space.

Given the normal estimation $\mathbf{N}$, we train a network $\boldsymbol{\epsilon}_{\theta}^{m}$ to predict a UV-coordinate image $\mathbf{C}$ conditioned on the body part segmentation $\mathbf{S}$. The pixel value of $\mathbf{C}$ is

$$ (8) $\mathbf{C}[x,y]=(u,v,\sigma)\;,$ $$

where $(u,v)$ is the predicted coordinate on the UV space of the panel for pixel $(x,y)$, $\sigma$ indicates whether it belongs to the front ($\sigma>0$) or the back ($\sigma<0$) panel. Through Eq. [8](https://arxiv.org/html/2504.08353v2#S4.E8), we establish the mapping from the pixel space to the UV space.

By assembling the results of UV and 3D mapping of Eq. [8](https://arxiv.org/html/2504.08353v2#S4.E8) and Eq. [7](https://arxiv.org/html/2504.08353v2#S4.E7), we can get a UV map $\tilde{\mathcal{U}}$, where

$$ (9) $\tilde{\mathcal{U}}[u,v]=P^{-1}(x,y,d)=(X,Y,Z)\;.$ $$

For the positions on $\tilde{\mathcal{U}}$ without projected points, we simply set their values to $\varnothing$. We also compute a mask $\tilde{\mathcal{M}}$ with
$\tilde{\mathcal{M}}[u,v]=1$ at where a pixel is projected, and $\tilde{\mathcal{M}}[u,v]=0$ otherwise. Due to occlusions, both $\tilde{\mathcal{U}}$ and $\tilde{\mathcal{M}}$ are incomplete. In the next section, we will complete them by fitting to the priors encoded in DISP.

##### Training.

We learn the mapping function in an image-to-image translation fashion with a conditional diffusion model $\boldsymbol{\epsilon}_{\theta}^{m}$. For the normal estimation $\mathbf{N}_{F}$ and $\mathbf{N}_{B}$, $\boldsymbol{\epsilon}_{\theta}^{m}$ is trained to predict their UV-coordinate image $\mathbf{C}_{F}$ and $\mathbf{C}_{B}$, and depth images $\mathbf{D}_{F}^{g}$ and $\mathbf{D}_{B}^{g}$ jointly.
The denoising process of $\boldsymbol{\epsilon}_{\theta}^{m}$ is conditioned on the estimated normals of the front and the back $\mathbf{N}=(\mathbf{N}_{F},\mathbf{N}_{B})$, the segmentation images $\mathbf{S}=(\mathbf{S}_{F},\mathbf{S}_{B})$, the body depth maps $\mathbf{D}=(\mathbf{D}_{F}^{b},\mathbf{D}_{B}^{b})$, and is learned by minimizing the loss

$$ (10) $\mathcal{L}=\mathbb{E}_{t,\mathbf{m}_{0},\boldsymbol{\epsilon}}\|\boldsymbol{ \epsilon}-\boldsymbol{\epsilon}_{\theta}^{m}\left(\sqrt{\bar{\alpha}_{t}} \mathbf{m}_{0}+\sqrt{1-\bar{\alpha}_{t}}\boldsymbol{\epsilon},\mathbf{N}, \mathbf{S},\mathbf{D},t\right)\|^{2}\;,$ $$

where $\mathbf{m}_{0}=[\mathbf{C}_{F},\mathbf{C}_{B},\mathbf{D}_{F}^{g},\mathbf{D}_{B
}^{g}]$. After the training, we assemble the results for both the front and the back to produce the UV map $\tilde{\mathcal{U}}$ and the mask $\tilde{\mathcal{M}}$. Compared with only using the front result, this provides more observations and constraints for the fitting, resulting in a reconstruction with higher quality for both the visible and invisible parts.

### 4.3. Fitting

The incomplete panel mask $\tilde{\mathcal{M}}$ and the incomplete UV map $\tilde{\mathcal{U}}$ provides partial information of the garment geometry and deformation, respectively. To recover a complete garment from them, we leverage the prior of DISP. We first recover the complete panel mask to recover the garment geometry in its rest shape, and then recover the complete UV map for the deformation. To remedy the synthetic-to-real domain gap and further improve the reconstruction accuracy, we rely on a post-optimization step to align the garment with image observations.

Figure: Figure 4. Recovering garment rest geometry. Given (a) the incomplete panel mask $\tilde{\mathcal{M}}$, we fit (b) the complete panel mask $\mathcal{M}$ by Eq. [11](https://arxiv.org/html/2504.08353v2#S4.E11). (c) shows the overlay of $\tilde{\mathcal{M}}$ in gray and $\mathcal{M}$ in white. (d) is the corresponding rest-state garment mesh $\bar{\mathbf{G}}$ for (b).
Refer to caption: x4.png

#### 4.3.1. Recovering the Rest Geometry.

To recover the garment rest geometry represented by the 2D panel shape from $\tilde{\mathcal{M}}$, we optimize the latent code $\mathbf{z}$ of Eq. [1](https://arxiv.org/html/2504.08353v2#S3.E1) so that its corresponding patterns match $\tilde{\mathcal{M}}$ as well as possible. The optimization objective is

$$ (11) $\mathcal{L}(\mathbf{z})=\sum\limits_{\scalebox{0.7}{$\mathbf{u}\in\mathcal{M}_ {+}$}}ReLU(s_{\mathbf{u}}(\mathbf{z}))-\lambda_{area}\sum\limits_{\scalebox{0. 7}{$\mathbf{u}\in\Omega$}}s_{\mathbf{u}}(\mathbf{z})+\lambda_{\mathbf{z}}|| \mathbf{z}||_{2}\;,$ $$

where $\mathcal{M}_{+}=\{\mathbf{u}|\tilde{\mathcal{M}}_{\mathbf{u}}=1,\mathbf{u}\in\Omega\}$, $s_{\mathbf{u}}(\mathbf{z})$ is the SDF value of $\mathbf{u}$ computed by ISP, and $\lambda_{area}$ and $\lambda_{\mathbf{z}}$ are the weighting constants. The first item in Eq. [11](https://arxiv.org/html/2504.08353v2#S4.E11) ensures that the projected UV points are within the panel, while the second one penalizes large panel area to make the panel contours surround the non-zero points of $\tilde{\mathcal{M}}$ as closely as possible. This optimization produces an optimal latent code $\mathbf{z}^{*}$ that we can use to infer a complete panel mask $\mathcal{M}$ and a rest-state garment mesh $\bar{\mathbf{G}}$ as shown in Fig. [4](https://arxiv.org/html/2504.08353v2#S4.F4).

#### 4.3.2. Recovering Deformed Geometry.

The diffusion model $\epsilon_{\theta}$ of DISP learns the distribution of plausible deformations represented by UV maps. To recover the full UV map $\mathcal{U}$, we use the partial UV map $\tilde{\mathcal{U}}$ and the recovered panel mask $\mathcal{M}$ as the manifold guidance (Chung et al., 2022a, b) in the reverse diffusion process of $\epsilon_{\theta}$:

$$ (12) $\displaystyle\nabla_{\mathbf{x}_{t}}\log p(\mathbf{x}_{t}|\tilde{\mathcal{U}}, \tilde{\mathcal{M}},\mathcal{M})$ $\displaystyle\simeq-\frac{\epsilon_{\theta}(\mathbf{x}_{t},t)}{\sigma_{t}}- \rho\nabla_{\mathbf{x}_{t}}\mathcal{L}(\hat{\mathbf{x}}_{0},\tilde{\mathcal{U} },\tilde{\mathcal{M}},\mathcal{M})\;\;,$ (13) $\displaystyle\hat{\mathbf{x}}_{0}$ $\displaystyle=\frac{1}{\sqrt{\bar{\alpha}_{t}}}\mathbf{x}_{t}-\sqrt{\frac{1- \bar{\alpha}_{t}}{\bar{\alpha}_{t}}}\epsilon_{\theta}(\mathbf{x}_{t},t)\;\;,$ $$

where $\rho$ is the guidance step size. $\mathcal{L}$ is the function that measures the difference between the generated and the given UV maps and panel masks

$$ (14) $\mathcal{L}(\hat{\mathbf{x}}_{0},\tilde{\mathcal{U}},\tilde{\mathcal{M}}, \mathcal{M})=\lVert\tilde{\mathcal{M}}*(\hat{\mathcal{U}}-\tilde{\mathcal{U}}) \rVert_{2}+\lVert\hat{\mathcal{M}}-\mathcal{M}\rVert_{1}\;,$ $$

where $\hat{\mathbf{x}}_{0}=[\hat{\mathcal{U}},\hat{\mathcal{M}}]$, $\hat{\mathcal{U}}$ and $\hat{\mathcal{M}}$ refer to the generated UV map and panel mask respectively, and $*$ denotes the element-wise multiplication. With the generated UV map $\hat{\mathcal{U}}$, we update the vertices of $\bar{\mathbf{G}}$ to get the recovered mesh $\mathbf{G}$ as shown in the bottom-left of Fig. [2](https://arxiv.org/html/2504.08353v2#S1.F2).

#### 4.3.3. Garment Refinement.

As the diffusion model learns the shape distribution from the garment simulation data which is generated with limited materials, external forces, and body motions, when handling in-the-wild images that are out-of-distribution, it can produce inaccurate UV maps by Eq. [12](https://arxiv.org/html/2504.08353v2#S4.E12) and result in inaccurate garment mesh that does not align with the images. To further improve the reconstruction accuracy, we refine the recovered mesh $\mathbf{G}$ in Sec. [4.3.2](https://arxiv.org/html/2504.08353v2#S4.SS3.SSS2) by optimizing its vertex positions to align it with the image observations. The loss function we use is

$$ (15) $\mathcal{L}=\lambda_{m}\mathcal{L}_{mask}+\lambda_{d}\mathcal{L}_{depth}+ \lambda_{n}\mathcal{L}_{normal}+\lambda_{u}\mathcal{L}_{uv}+\lambda_{p} \mathcal{L}_{phys},$ $$

where $\lambda_{m}$, $\lambda_{d}$, $\lambda_{n}$, $\lambda_{u}$ and $\lambda_{p}$ are the weighting scalars. $\mathcal{L}_{mask}$, $\mathcal{L}_{depth}$ and $\mathcal{L}_{normal}$ penalize the difference between the rendered front and back masks, depth and normal of the mesh $\mathbf{G}$ and their corresponding estimation respectively. $\mathcal{L}_{uv}$ ensures the corresponding UV map $\hat{\mathcal{U}}$ of $\mathbf{G}$ aligns with the partial UV observations $\tilde{\mathcal{U}}$ by

$$ (16) $\mathcal{L}_{uv}=\lVert\tilde{\mathcal{M}}*(\hat{\mathcal{U}}-\tilde{\mathcal{ U}})\rVert_{2}.$ $$

$\mathcal{L}_{phys}$ contains a set of physics-based mesh regularization (Narain et al., 2012; Santesteban et al., 2022) computed by using the recovered rest-state garment $\bar{\mathbf{G}}$ as the reference

$$ (17) $\mathcal{L}_{phys}=\mathcal{L}_{strain}+\mathcal{L}_{bend}+\mathcal{L}_{ gravity}+\mathcal{L}_{collision},$ $$

where $\mathcal{L}_{strain}$ is the membrane strain energy caused by the deformation, $\mathcal{L}_{bend}$ is the bending energy resulting from the folding of adjacent faces, $\mathcal{L}_{gravity}$ is the gravitational potential energy and $\mathcal{L}_{collision}$ is the penalty for body garment collision.

However, directly optimizing the vertex positions using Eq. [15](https://arxiv.org/html/2504.08353v2#S4.E15) will lead to a suboptimal solution, as vertices are not strongly coupled.
Inspired by (Ulyanov et al., 2018; Gadelha et al., 2021), we introduce a multi-layer perceptron (MLP) for the optimization to update vertices by a neural displacement field. To be specific, we initialize an MLP network $f$ with learnable parameters $\phi$. Given the vertex $V$ of mesh $\mathbf{G}$ and its canonical position $\bar{V}$ on $\bar{\mathbf{G}}$, the network $f_{\phi}$ predicts its displacement by

$$ (18) $\Delta V=f_{\phi}(V,\bar{V}).$ $$

We use the updated vertex position $V+\Delta V$ to compute the loss of Eq. [15](https://arxiv.org/html/2504.08353v2#S4.E15) and compute the gradient with respect to $\phi$ for its learning. Since neural networks tend to learn low-frequency functions (Rahaman et al., 2019), the result after this step is a bit smooth. To further recover fine surface details, we perform an additional refinement step by directly optimizing the garment mesh vertices with Eq. [15](https://arxiv.org/html/2504.08353v2#S4.E15).

Figure: Figure 5. Body refinement. For the image of (a), we refine the initial body estimation of (c) by Eq. [19](https://arxiv.org/html/2504.08353v2#S4.E19) to improve its accuracy and align it with the image as (b).
Refer to caption: x5.png

#### 4.3.4. Refining Body

As the body estimation can be inaccurate, making it inconsistent with the garment recovery as shown in Fig.[5](https://arxiv.org/html/2504.08353v2#S4.F5), we refine the SMPL body parameters $\beta$ and $\theta$ by minimizing

$$ (19) $\mathcal{L}(\beta,\theta)=\lambda_{m}\mathcal{L}_{mask}+\lambda_{o}\mathcal{L} _{order}+\lambda_{j}\mathcal{L}_{joint}+\lambda_{\beta}||\beta||^{2}_{2},$ $$

where $\lambda_{m}$, $\lambda_{o}$, $\lambda_{j}$, and $\lambda_{\beta}$ are the weighting scalars. $\mathcal{L}_{mask}$ penalizes the difference between the rendered body mask with segmented mask. $\mathcal{L}_{joint}$ penalizes the difference between 2D projection of body joints and the detected 2D joints. $\mathcal{L}_{order}$ ensures that the body mesh is inside the garment mesh by enforcing the rendered depth of the body is larger than the depth of the garments $\mathbf{D}_{F}^{g}$ and $\mathbf{D}_{B}^{g}$ estimated in Sec. [4.2](https://arxiv.org/html/2504.08353v2#S4.SS2)

$$ (20) $\mathcal{L}_{order}=||ReLU(\mathbf{D}_{F}^{g}-\mathbf{D}_{F}^{b}+\delta)||_{1} +||ReLU(\mathbf{D}_{B}^{g}-\mathbf{D}_{B}^{b}+\delta)||_{1},$ $$

where $\mathbf{D}_{F}^{b}$ and $\mathbf{D}_{B}^{b}$ are the rendered front and back depth images of the body mesh and $\delta$ is the threshold value. Note that we first refine the body with the garment depth estimations $\mathbf{D}_{F}^{g}$ and $\mathbf{D}_{B}^{g}$, and then optimize the garment with the refined body as introduced in Sec. [4.3.3](https://arxiv.org/html/2504.08353v2#S4.SS3.SSS3).

## 5. Experiments

### 5.1. Dataset and Evaluation Metrics

Due to the intricate structure of garments, collecting real 3D data with complete geometry for them is extremely difficult. Instead, we use physics-based simulation to generate garment with realistic deformations in its interaction with the underlying body.

**Table 1. Quantitative comparisons. Our method outperforms SMPLicit, DrapeNet, ISP and GaRec in terms of CD, IoU and NC on different garment categories. The unit of CD is cm.**
| Skirt<br>CD $\downarrow$<br>IoU $\uparrow$<br>NC $\uparrow$<br>SMPLicit<br>3.00<br>65.03<br>0.02<br>DrapeNet<br>n/a<br>n/a<br>n/a<br>ISP<br>2.51<br>71.10<br>0.76<br>GaRec<br>2.00<br>93.81<br>0.80<br>Ours<br>1.21<br>95.32<br>0.83 | Trousers<br>CD $\downarrow$<br>IoU $\uparrow$<br>NC $\uparrow$<br>SMPLicit<br>1.59<br>68.19<br>-0.03<br>DrapeNet<br>1.38<br>74.23<br>0.84<br>ISP<br>1.53<br>57.745<br>0.85<br>GaRec<br>1.12<br>88.52<br>0.86<br>Ours<br>0.74<br>94.00<br>0.88 |
| --- | --- |
| Shirt<br>CD $\downarrow$<br>IoU $\uparrow$<br>NC $\uparrow$<br>SMPLicit<br>6.53<br>47.50<br>0.06<br>DrapeNet<br>1.93<br>77.15<br>0.84<br>ISP<br>1.92<br>68.64<br>0.83<br>GaRec<br>1.20<br>93.20<br>0.83<br>Ours<br>0.85<br>94.02<br>0.89 | Open Shirt<br>CD $\downarrow$<br>IoU $\uparrow$<br>NC $\uparrow$<br>SMPLicit<br>2.37<br>61.27<br>-0.08<br>DrapeNet<br>1.76<br>73.56<br>0.78<br>ISP<br>1.90<br>69.27<br>0.77<br>GaRec<br>1.46<br>92.81<br>0.73<br>Ours<br>1.19<br>92.48<br>0.82 |

CLOTH3D (Bertiche et al., 2020) is a synthetic dataset, with 3D garments draped on T-posed SMPL bodies (Loper and Black, 2014). For each clothing category, including shirt, open shirt, skirt and trousers, we randomly select 33 samples. Each pair of garment and body models is simulated with the motion data sourced from the dance category of the AMASS dataset (Mahmood et al., 2019). The motion sequences are generated by using Blender (blender, 2018) and Marvelous Designer (Designer, 2018). Additional pins are manually set for open shirt, skirt and trousers to avoid sliding during the simulation. For each body sample in the sequence, we randomly rotate it and render its front and back body part segmentation and depth images. For the corresponding garment sample, we rotate it with the same angle and render its front and back normal and depth images. Its front and back UV coordinate images are generated using the UV parameterization of ISP. For each garment category, we randomly select 30 pairs of garment and body for training and use the rest pairs for the evaluation.

To evaluate the quality of garment reconstruction, we use the Chamfer Distance (CD) and the Normal Consistency (NC) between the ground truth and the recovered garment mesh, and the Intersection over Union (IoU) between the ground truth mask and the rendered mask of reconstructed garment mesh. The quantitative comparison is conducted on the test set of the synthetic data, and the qualitative evaluation is conducted on in-the-wild images.

### 5.2. Results

In Table [1](https://arxiv.org/html/2504.08353v2#S5.T1), we present the quantitative comparison between our method and the state-of-the-art approaches: SMPLicit (Corona et al., 2021), DrapeNet (DeLuigi et al., 2023), ISP (Li et al., 2023), and GaRec (Li et al., 2024b). Our method achieves significantly better performance across all garment categories—Skirt, Trousers, Shirt, and Open Shirt—in terms of CD, IoU and NC.

Fig. [6](https://arxiv.org/html/2504.08353v2#Sx1.F6) provides a qualitative comparison of the results reconstructed from the in-the-wild image. Methods like BCNet (Jiang et al., 2020), SMPLicit, and ISP rely solely on the SMPL body model’s skinning function to deform garments, which limits them to generating results tightly adhered to the body surface. ECON (Xiu et al., 2023) and GaRec, on the other hand, can recover garments that stand away from the body. However, ECON produces a single watertight mesh that models both the body and garment as a single entity. While GaRec generates standalone garment surfaces, its recovered meshes appear overly flat and lack realistic folds or creases.
In contrast, our method, leveraging the proposed fitting approach that incorporates back normal estimation and DISP priors, faithfully reconstructs garment meshes with high-fidelity wrinkle details, both on the front and back of the garment. Fig. [10](https://arxiv.org/html/2504.08353v2#Sx1.F10) and [11](https://arxiv.org/html/2504.08353v2#Sx1.F11) provides more results of our method applied to in-the-wild images, demonstrating its ability to produce realistic 3D meshes with fine details for both tight-fitting and loose-fitting garments.

### 5.3. Ablation Study

Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7) presents the ablation study of our fitting method. As shown in Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(c), the initial reconstruction, without the post-refinement step described in Section [4.3.3](https://arxiv.org/html/2504.08353v2#S4.SS3.SSS3), fails to fully align with the input image shown in Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(a). Incorporating a neural displacement field to optimize the initial mesh improves reconstruction accuracy, as seen in Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(d). Further refinement by directly optimizing vertex positions enhances the wrinkle details, as illustrated in Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(b). However, applying post-refinement without first optimizing the neural displacement field (Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(e)) struggles to recover an accurate shape, as each vertex is optimized independently, leading to suboptimal results. Finally, Fig. [7](https://arxiv.org/html/2504.08353v2#Sx1.F7)(f) shows the outcome when only the front normal estimation is used throughout the fitting process described in Section [4.3](https://arxiv.org/html/2504.08353v2#S4.SS3). The lack of constraints for the back surface results in unrealistic deformations on the back. The corresponding quantitative results are provided in the supplementary materials.

### 5.4. Downstream Applications

##### Retargeting

Since our method produces separate models for the garment and the underlying body, we can easily repose it on the new body. In Fig. [9](https://arxiv.org/html/2504.08353v2#Sx1.F9), we show the retargeting results for the reconstructed open shirt and trousers by transferring them onto bodies with different poses and shapes. Accurate reconstruction of garments results in realistic retargeting.

##### Texture Editing

Since we reconstruct both the 3D model and the corresponding 2D panels for garment, we can easily realize texture editing. As shown in Fig. [9](https://arxiv.org/html/2504.08353v2#Sx1.F9), by painting patterns or drawing specific figures onto the recovered panels, the mesh will show the texture on the corresponding position.

## 6. Conclusion

We have presented a novel approach for recovering realistic 3D garment meshes from monocular images. Our method leverages Implicit Sewing Patterns (ISP) and a generative diffusion model to learn plausible garment shape priors defined in a 2D UV space. By utilizing diffusion schemes, we complement 2D observations for the occluded parts of the garments and lift them into 3D space. Additionally, we design a diffusion-based mapping across 2D, 3D, and UV space, enabling the alignment of learned priors with image observations to produce accurate 3D garment reconstructions. Our method outperforms existing approaches across different types of garments, and the resulting reconstructions are readily applicable to downstream tasks, such as garment retargeting and texture editing.

##### Limitations

While our method is capable of producing realistic 3D reconstructions for a wide variety of garments, it has certain limitations.
As shown in the middle example of the fourth row in Fig. [10](https://arxiv.org/html/2504.08353v2#Sx1.F10), it is challenging for our method to capture very small wrinkles. This limitation arises because our normal loss relies on a differentiable renderer (Ravi et al., 2020), which uses interpolation and approximations for normal and gradient computation. These approximations tend to smooth out high-frequency geometric details. Additionally, since small wrinkles contribute only marginally to the overall loss, their gradients can be overwhelmed during optimization. Together, these factors explain the observed results.
Beside, our method cannot currently handle garments with multi-layered structures, such as ruffle-layered skirts. A potential solution could involve incorporating additional panels into ISP to support layered designs. Furthermore, our approach requires full-body images of clothed individuals and, therefore, cannot handle images with partial garments or profile views. Due to the inherent ill-posed nature of 3D reconstruction from a single image, our method also cannot address depth ambiguity and does not fully capture the physical behavior of garments. The result for video input can also seem jittery. In future work, we aim to address this problem by modeling garment deformations over time using video sequences.

## Acknowledgments

This work was supported in part by the Swiss National Science Foundation grant and by the Metaverse Center Grant from the MBZUAI Research Office.

## References

- (1)
- Alldieck et al. (2019a)
T. Alldieck, M. Magnor, B. Bhatnagar, C. Theobalt, and G. Pons-Moll. 2019a.
Learning to reconstruct people in clothing from a single RGB camera. In *Conference on Computer Vision and Pattern Recognition*. 1175–1186.
- Alldieck et al. (2019b)
T. Alldieck, G. Pons-Moll, C. Theobalt, and M. Magnor. 2019b.
Tex2shape: Detailed full human body geometry from a single image. In *International Conference on Computer Vision*. 2293–2303.
- Alldieck et al. (2022)
T. Alldieck, M. Zanfir, and C. Sminchisescu. 2022.
Photorealistic Monocular 3D Reconstruction of Humans Wearing Clothing. In *Conference on Computer Vision and Pattern Recognition*. 1506–1515.
- Anciukevičius et al. (2023)
T. Anciukevičius, Z. Xu, M. Fisher, P. Henderson, H. Bilen, N. J. Mitra, and P. Guerrero. 2023.
Renderdiffusion: Image Diffusion for 3D Reconstruction, Inpainting and Generation. In *Conference on Computer Vision and Pattern Recognition*. 12608–12618.
- Bae and Davison (2024)
G. Bae and A. Davison. 2024.
Rethinking Inductive Biases for Surface Normal Estimation. In *Conference on Computer Vision and Pattern Recognition*. 9535–9545.
- Bertiche et al. (2020)
H. Bertiche, M. Madadi, and S. Escalera. 2020.
CLOTH3D: Clothed 3D Humans. In *European Conference on Computer Vision*. 344–359.
- Bhatnagar et al. (2019)
B. L. Bhatnagar, G. Tiwari, C. Theobalt, and G. Pons-Moll. 2019.
Multi-Garment Net: Learning to Dress 3D People from Images. In *International Conference on Computer Vision*.
- blender (2018)
blender 2018.
Blender.
https://www.blender.org/.
- Bogo et al. (2016)
F. Bogo, A. Kanazawa, C. Lassner, P. Gehler, J. Romero, and M. J. Black. 2016.
Keep It SMPL: Automatic Estimation of 3D Human Pose and Shape from a Single Image. In *European Conference on Computer Vision*.
- Casado-Elvira et al. (2022)
A. Casado-Elvira, M. Trinidad, and D. Casas. 2022.
PERGAMO: Personalized 3d Garments from Monocular Video. In *Computer Graphics Forum*, Vol. 41. 293–304.
- Chung et al. (2022a)
H. Chung, J. Kim, M. T. Mccann, M. L. Klasky, and J. C. Ye. 2022a.
Diffusion Posterior Sampling for General Noisy Inverse Problems. In *International Conference on Learning Representations*.
- Chung et al. (2022b)
H. Chung, B. Sim, D. Ryu, and J. C. Ye. 2022b.
Improving Diffusion Models for Inverse Problems Using Manifold Constraints. In *Advances in Neural Information Processing Systems*. 25683–25696.
- Corona et al. (2021)
E. Corona, A. Pumarola, G. Alenya, G. Pons-Moll, and F. Moreno-Noguer. 2021.
Smplicit: Topology-Aware Generative Model for Clothed People. In *Conference on Computer Vision and Pattern Recognition*.
- Danerek et al. (2017)
R. Danerek, E. Dibra, C. öztireli, R. Ziegler, and M. Gross. 2017.
Deepgarment : 3D Garment Shape Estimation from a Single Image.
*Eurographics* (2017).
- DeLuigi et al. (2023)
L. DeLuigi, R. Li, B. Guillard, M. Salzmann, and P. Fua. 2023.
Drapenet: Generating Garments and Draping Them with Self-Supervision. In *Conference on Computer Vision and Pattern Recognition*.
- Designer (2018)
M. Designer. 2018.
[https://www.marvelousdesigner.com](https://www.marvelousdesigner.com).
- Dhariwal and Nichol (2021)
Prafulla Dhariwal and Alexander Nichol. 2021.
Diffusion Models Beat GANs on Image Synthesis. In *Advances in Neural Information Processing Systems*, Vol. 34. 8780–8794.
- Gadelha et al. (2021)
M. Gadelha, R. Wang, and S. Maji. 2021.
Deep Manifold Prior. In *International Conference on Computer Vision*. 1107–1116.
- Georgakis et al. (2020)
G. Georgakis, R. Li, S. Karanam, T. Chen, J. Košecká, and Z. Wu. 2020.
Hierarchical Kinematic Human Mesh Recovery. In *European Conference on Computer Vision*. 768–784.
- Goel et al. (2023)
S. Goel, G. Pavlakos, J. Rajasegaran, A. Kanazawa, and J. Malik. 2023.
Humans in 4D: Reconstructing and Tracking Humans with Transformers. In *International Conference on Computer Vision*.
- Guillard et al. (2022)
B. Guillard, F. Stella, and P. Fua. 2022.
MeshUDF: Fast and Differentiable Meshing of Unsigned Distance Field Networks. In *European Conference on Computer Vision*.
- Guo et al. (2024)
J. Guo, F. Prada, D. Xiang, J. Romero, C. Wu, H. S. Park, T. Shiratori, and S. Saito. 2024.
Diffusion Shape Prior for Wrinkle-Accurate Cloth Registration. In *International Conference on 3D Vision*.
- He et al. (2020)
T. He, J. Collomosse, H. Jin, and S. Soatto. 2020.
Geo-PIFu: Geometry and pixel aligned implicit functions for single-view human reconstruction.
*Advances in Neural Information Processing Systems* 33 (2020), 9276–9287.
- He et al. (2021)
T. He, Y. Xu, S. Saito, S. Soatto, and T. Tung. 2021.
ARCH++: Animation-Ready Clothed Human Reconstruction Revisited. In *International Conference on Computer Vision*. 11046–11056.
- Ho et al. (2020)
J. Ho, A. Jain, and P. Abbeel. 2020.
Denoising Diffusion Probabilistic Models.
*Advances in Neural Information Processing Systems* 33 (2020), 6840–6851.
- Huang et al. (2020)
Z. Huang, Y. Xu, C. Lassner, H. Li, and T. Tung. 2020.
Arch: Animatable Reconstruction of Clothed Humans. In *Conference on Computer Vision and Pattern Recognition*. 3093–3102.
- Jackson et al. (2018)
A. Jackson, C. Manafas, and G. Tzimiropoulos. 2018.
3D Human Body Reconstruction from A Single Image via Volumetric Regression. In *European Conference on Computer Vision Workshops*.
- Jiang et al. (2020)
B. Jiang, J. Zhang, Y. Hong, J. Luo, L. Liu, and H. Bao. 2020.
Bcnet: Learning Body and Cloth Shape from a Single Image. In *European Conference on Computer Vision*.
- Kanazawa et al. (2018)
A. Kanazawa, M. J. Black, D. W. Jacobs, and J. Malik. 2018.
End-To-End Recovery of Human Shape and Pose. In *Conference on Computer Vision and Pattern Recognition*.
- Khirodkar et al. (2025)
R. Khirodkar, T. Bagautdinov, J. Martinez, S. Zhaoen, A. James, P. Selednik, S Anderson, and S. Saito. 2025.
Sapiens: Foundation for Human Vision Models. In *European Conference on Computer Vision*. 206–228.
- Kirillov et al. (2023)
A. Kirillov, E. Mintun, N. Ravi, H. Mao, C. Rolland, L. Gustafson, T. Xiao, S. Whitehead, A. C. Berg, W.-Y. Lo, P. Dollár, and R. Girshick. 2023.
Segment Anything. In *arXiv Preprint*.
- Kolotouros et al. (2019)
N. Kolotouros, G. Pavlakos, M. J. Black, and K. Daniilidis. 2019.
Learning to Reconstruct 3D Human Pose and Shape via Model-Fitting in the Loop. In *Conference on Computer Vision and Pattern Recognition*.
- Lassner et al. (2017)
C. Lassner, J. Romero, M. Kiefel, F. Bogo, M.J. Black, and P.V. Gehler. 2017.
Unite the People: Closing the Loop Between 3D and 2D Human Representations. In *Conference on Computer Vision and Pattern Recognition*.
- Li et al. (2021a)
J. Li, C. Xu, Z. Chen, S. Bian, L. Yang, and C. Lu. 2021a.
Hybrik: A Hybrid Analytical-Neural Inverse Kinematics Solution for 3D Human Pose and Shape Estimation. In *Conference on Computer Vision and Pattern Recognition*.
- Li et al. (2020)
P. Li, Y. Xu, Y. Wei, and Y. Yang. 2020.
Self-Correction for Human Parsing.
*IEEE Transactions on Pattern Analysis and Machine Intelligence* 44, 6 (2020), 3260–3271.
- Li et al. (2024a)
R. Li, C. Dumery, Z. Dang, and P. Fua. 2024a.
Reconstruction of Manipulated Garment with Guided Deformation Prior. In *Advances in Neural Information Processing Systems*.
- Li et al. (2024b)
R. Li, C. Dumery, B. Guillard, and P. Fua. 2024b.
Garment Recovery with Shape and Deformation Priors. In *Conference on Computer Vision and Pattern Recognition*.
- Li et al. (2023)
R. Li, B. Guillard, and P. Fua. 2023.
ISP: Multi-Layered Garment Draping with Implicit Sewing Patterns. In *Advances in Neural Information Processing Systems*.
- Li et al. (2022)
R. Li, B. Guillard, E. Remelli, and P. Fua. 2022.
DIG: Draping Implicit Garment over the Human Body. In *Asian Conference on Computer Vision*.
- Li et al. (2021b)
R. Li, M. Zheng, S. Karanam, T. Chen, and Z. Wu. 2021b.
Everybody Is Unique: Towards Unbiased Human Mesh Recovery. In *British Machine Vision Conference*.
- Li et al. (2024c)
W. Li, M. Liu, H. Liu, B. Ren, X. Li, Y. You, and N. Sebe. 2024c.
HYRE: Hybrid Regressor for 3D Human Pose and Shape Estimation.
*IEEE Transactions on Image Processing* (2024).
- Liu et al. (2008)
L. Liu, L. Zhang, Y. Xu, C. Gotsman, and S. J. Gortler. 2008.
A Local/global Approach to Mesh Parameterization. In *Proceedings of the Symposium on Geometry Processing*. 1495–1504.
- Liu et al. (2023b)
Ruoshi Liu, Rundi Wu, Basile Van Hoorick, Pavel Tokmakov, Sergey Zakharov, and Carl Vondrick. 2023b.
Zero-1-To-3: Zero-Shot One Image to 3D Object. In *International Conference on Computer Vision*. 9298–9309.
- Liu et al. (2023a)
X. Liu, J. Li, and G. Lu. 2023a.
Modeling Realistic Clothing from a Single Image Under Normal Guide.
*IEEE Transactions on Visualization and Computer Graphics* (2023).
- Loper and Black (2014)
M. Loper and M.J. Black. 2014.
Opendr: An Approximate Differentiable Renderer. In *European Conference on Computer Vision*. 154–169.
- Loper et al. (2015)
M. Loper, N. Mahmood, J. Romero, G. Pons-Moll, and M.J. Black. 2015.
SMPL: A Skinned Multi-Person Linear Model.
*ACM SIGGRAPH Asia* 34, 6 (2015).
- Luo et al. (2024)
Z. Luo, H. Liu, C. Li, W. Du, Z. Jin, W. Sun, Y. Nie, W. Chen, and X. Han. 2024.
GarVerseLOD: High-Fidelity 3D Garment Reconstruction from a Single In-the-Wild Image using a Dataset with Levels of Details.
*ACM Transactions on Graphics* 43, 6 (2024), 1–12.
- Ma et al. (2022)
Q. Ma, J. Yang, M. J. Black, and S. Tang. 2022.
Neural Point-Based Shape Modeling of Humans in Challenging Clothing. In *International Conference on 3D Vision*.
- Mahmood et al. (2019)
N. Mahmood, N. Ghorbani, N. F. Troje, G. Pons-Moll, and M. J. Black. 2019.
AMASS: Archive of Motion Capture as Surface Shapes. In *International Conference on Computer Vision*. 5442–5451.
- Melas-Kyriazi et al. (2023)
L. Melas-Kyriazi, C. Rupprecht, and A. Vedaldi. 2023.
Pc2: Projection-Conditioned Point Cloud Diffusion for Single-Image 3D Reconstruction. In *Conference on Computer Vision and Pattern Recognition*. 12923–12932.
- Moon and Lee (2020)
G. Moon and K.M. Lee. 2020.
I2l-Meshnet: Image-To-Lixel Prediction Network for Accurate 3D Human Pose and Mesh Estimation from a Single RGB Image. In *European Conference on Computer Vision*.
- Moon et al. (2022)
G. Moon, H. Nam, T. Shiratori, and K.M. Lee. 2022.
3D Clothed Human Reconstruction in the Wild. In *European Conference on Computer Vision*.
- Müller et al. (2023)
N. Müller, Y. Siddiqui, L. Porzi, S. R. Bulo, P. Kontschieder, and M. Nießner. 2023.
Diffrf: Rendering-Guided 3D Radiance Field Diffusion. In *Conference on Computer Vision and Pattern Recognition*. 4328–4338.
- Narain et al. (2012)
R. Narain, A. Samii, and J.F. O’brien. 2012.
Adaptive anisotropic remeshing for cloth simulation.
*ACM Transactions on Graphics* (2012).
- Pavlakos et al. (2019)
G. Pavlakos, V. Choutas, N. Ghorbani, T. Bolkart, A. Osman, D. Tzionas, and M.J. Black. 2019.
Expressive Body Capture: 3D Hands, Face, and Body from a Single Image. In *Conference on Computer Vision and Pattern Recognition*.
- Pietroni et al. (2022)
N. Pietroni, C. Dumery, R. Falque, M. Liu, T. Vidal-Calleja, and O. Sorkine-Hornung. 2022.
Computational Pattern Making from 3D Garment Models.
*ACM Transactions on Graphics* 41, 4 (2022), 1–14.
- Poole et al. (2022)
Ben Poole, A. Jain, J. T. Barron, and Ben Mildenhall. 2022.
Dreamfusion: Text-To-3D Using 2D Diffusion. In *International Conference on Learning Representations*.
- Rahaman et al. (2019)
N. Rahaman, A. Baratin, D. Arpit, F. Draxler, M. Lin, F. Hamprecht, Y. Bengio, and A. Courville. 2019.
On the Spectral Bias of Neural Networks. In *International Conference on Machine Learning*. 5301–5310.
- Ravi et al. (2024)
N. Ravi, V. Gabeur, Y. Hu, R. Hu, C. Ryali, T. Ma, H. Khedr, R. Rädle, C. Rolland, Laura L. Gustafson, and Others. 2024.
Sam 2: Segment Snything in Images and Videos.
*arXiv Preprint* (2024).
- Ravi et al. (2020)
Nikhila Ravi, Jeremy Reizenstein, David Novotny, Taylor Gordon, Wan-Yen Lo, Justin Johnson, and Georgia Gkioxari. 2020.
PyTorch3D.
[https://github.com/facebookresearch/pytorch3d](https://github.com/facebookresearch/pytorch3d).
- Rombach et al. (2022)
Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, and Björn Ommer. 2022.
High-resolution image synthesis with latent diffusion models. In *Conference on Computer Vision and Pattern Recognition*. 10684–10695.
- Saito et al. (2019)
S. Saito, Z. Huang, R. Natsume, S. Morishima, A. Kanazawa, and H. Li. 2019.
PIFu: Pixel-Aligned Implicit Function for High-Resolution Clothed Human Digitization. In *International Conference on Computer Vision*.
- Saito et al. (2020)
S. Saito, T. Simon, J. Saragih, and H. Joo. 2020.
Pifuhd: Multi-Level Pixel-Aligned Implicit Function for High-Resolution 3D Human Digitization. In *Conference on Computer Vision and Pattern Recognition*. 84–93.
- Santesteban et al. (2022)
I. Santesteban, M.A. Otaduy, and D. Casas. 2022.
SNUG: Self-Supervised Neural Dynamic Garments. In *Conference on Computer Vision and Pattern Recognition*.
- Song et al. (2021)
J. Song, C. Meng, and S. Ermon. 2021.
Denoising Diffusion Implicit Models. In *International Conference on Learning Representations*.
- Srivastava et al. (2022)
A. Srivastava, C. Pokhariya, S. Jinka, and A. Sharma. 2022.
xCloth: Extracting Template-free Textured 3D Clothes from a Monocular Image. In *Proceedings of the 30th ACM International Conference on Multimedia*. 2504–2512.
- Stathopoulos et al. (2024)
A. Stathopoulos, L. Han, and D. Metaxas. 2024.
Score-Guided Diffusion for 3D Human Recovery. In *Conference on Computer Vision and Pattern Recognition*. 906–915.
- Tyszkiewic et al. (2023)
M. Tyszkiewic, P. Fua, and E. Trulls. 2023.
GECCO: Geometrically-Conditioned Point Diffusion Models. In *International Conference on Computer Vision*.
- Ulyanov et al. (2018)
D. Ulyanov, A. Vedaldi, and V. Lempitsky. 2018.
Deep Image Prior. In *Conference on Computer Vision and Pattern Recognition*. 9446–9454.
- Xiu et al. (2023)
Y. Xiu, J. Yang, X. Cao, D. Tzionas, and M. J Black. 2023.
ECON: Explicit Clothed humans Optimized via Normal integration. In *Conference on Computer Vision and Pattern Recognition*. 512–523.
- Xiu et al. (2022)
Y. Xiu, J. Yang, D. Tzionas, and M. J Black. 2022.
ICON: Implicit clothed humans obtained from normals. In *Conference on Computer Vision and Pattern Recognition*. 13286–13296.
- Xu et al. (2024a)
J. Xu, Y. Guo, and Y. Peng. 2024a.
FinePOSE: Fine-Grained Prompt-Driven 3D Human Pose Estimation via Diffusion Models. In *Conference on Computer Vision and Pattern Recognition*. 561–570.
- Xu et al. (2024b)
Yinghao Xu, Hao Tan, Fujun Luan, Sai Bi, Peng Wang, Jiahao Li, Zifan Shi, Kalyan Sunkavalli, Gordon Wetzstein, Zexiang Xu, et al. 2024b.
Dmv3D: Denoising Multi-View Diffusion Using 3d Large Reconstruction Model. In *International Conference on Learning Representations*.
- Yang et al. (2018)
S. Yang, Z. Pan, T. Amert, K. Wang, L. Yu, T. Berg, and M. Lin. 2018.
Physics-Inspired Garment Recovery from a Single-View Image.
*ACM Transactions on Graphics* 37, 5 (2018), 1–14.
- Zakharkin et al. (2021)
I. Zakharkin, K. Mazur, A. Grigorev, and V. Lempitsky. 2021.
Point-Based Modeling of Human Clothing. In *International Conference on Computer Vision*.
- Zheng et al. (2021)
Z. Zheng, T. Yu, Y. Liu, and Q. Dai. 2021.
Pamir: Parametric model-conditioned implicit representation for image-based human reconstruction.
*IEEE Transactions on Pattern Analysis and Machine Intelligence* 44, 6 (2021), 3170–3184.
- Zheng et al. (2019)
Z. Zheng, T. Yu, Y. Wei, Q. Dai, and Y. Liu. 2019.
DeepHuman: 3D Human Reconstruction from A Single Image. In *International Conference on Computer Vision*. 7739–7749.
- Zhu et al. (2020)
H. Zhu, Y. Cao, H. Jin, W. Chen, D. Du, Z. Wang, S. Cui, and X. Han. 2020.
Deep Fashion3D: A Dataset and Benchmark for 3d Garment Reconstruction from Single Images. In *European Conference on Computer Vision*. 512–530.
- Zhu et al. (2022)
H. Zhu, L. Qiu, Y. Qiu, and X. Han. 2022.
Registering Explicit to Implicit: Towards High-Fidelity Garment Mesh Reconstruction from Single Images. In *Conference on Computer Vision and Pattern Recognition*. 3845–3854.