<!-- markdownlint-disable -->
## Contents
- 1 Introduction
- 2 Related Work
- 3 Method
  - 3.1 Preliminaries on Sewing Pattern
  - 3.2 Extended Representation for Sewing Pattern
  - 3.3 Compact Latent for Sewing Pattern
  - 3.4 Multimodal Conditions of Diffusion Model
- 4 Experiments
  - 4.1 Experiment Setup
  - 4.2 Qualitative Comparison
  - 4.3 Quantitative Comparison
  - 4.4 Ablation Study
- 5 Conclusion and Limitations
- References
- 6 Representation Details
- 7 Comparison with Parametric Method
- 8 Concurrent Work Comparison
- 9 Dataset Annotation Details
- 10 Extended Vector v.s Origin Vector
- 11 One-step Training v.s Two-step Training
- 12 User Study Details
- 13 Use Case
- 14 Qualitative Results

## Abstract

Abstract Generating sewing patterns in garment design is receiving increasing attention due to its CG-friendly and flexible-editing nature.
Previous sewing pattern generation methods have been able to produce exquisite clothing, but struggle to design complex garments with detailed control.
To address these issues, we propose SewingLDM , a multi-modal generative model that generates sewing patterns controlled by text prompts, body shapes, and garment sketches.
Initially, we extend the original vector of sewing patterns into a more comprehensive representation to cover more intricate details and then compress them into a compact latent space.
To learn the sewing pattern distribution in the latent space, we design a two-step training strategy to inject the multi-modal conditions, i.e . , body shapes, text prompts, and garment sketches, into a diffusion model, ensuring the generated garments are body-suited and detail-controlled.
Comprehensive qualitative and quantitative experiments show the effectiveness of our proposed method, significantly surpassing previous approaches in terms of complex garment design and various body adaptability.
Our project page: https://shengqiliu1.github.io/SewingLDM .

## 1 Introduction

Clothes play a pivotal role in shaping human aesthetics and physique, where appropriate clothes can beautify their overall appearance and highlight human physical attributes.
Therefore, garment design has always been a crucial component that
significantly impacts both digital character creation and real-life human visual presentation.
In recent years, many garment generation methods [55, 30, 27, 21, 34, 28, 56, 4, 44, 45, 39, 10, 65, 75, 26] have emerged to generate desired garments for users under various conditions.

Typically, garment generation can be classified into 2D and 3D methods.
The 2D generation methods [4, 44, 45, 26] can produce visually appealing results but cannot maintain consistency across different views, failing to drape on human bodies.
Therefore, many recent works are focusing on 3D cloth generation [55, 30].
Although these methods can generate high-quality meshes or neural fields, they pose the challenge of clipping between clothes and bodies when draping clothes onto the human body, and they are incompatible with the digital garment production pipeline.
Meanwhile, the sewing pattern is a more widely used representation for garments in the industry because it facilitates both physical simulation and animation in CG-friendly fashions [3, 8].
Previous methods for sewing pattern generation [27, 21, 34] achieve fantastic garment generation but fall short in designing complex features.
Most recently, concurrent works [7, 46, 76] introduce complex features with reduced tokenization, leading to significant advancements in garment generation.
However, these methods typically ignore human body shapes, preventing the creation of made-to-measure garments.
Apart from learning-based methods, the parametric garment design tool [28] allows users to model complex garments and considers relations with body shapes.
Nonetheless, this tool requires pre-defined templates and detailed body measurements, and then with a delicate selection of control parameters to generate the garments, which require professional knowledge of garment designs.
In summary, there are two main challenges in suitable sewing pattern generation: 1) Designing a general representation for complex sewing patterns, and 2) Enabling control over garment details and ensuring garments are body-suited.

To address these issues, we design a novel architecture named SewingLDM for generating complex sewing patterns under the control of texts, body shapes, and garment sketches.
To represent complex designs of sewing patterns, we especially design an extended representation to encompass intricate types of edges and attachments of garments, enabling more general and complex garment learning.
Subsequently, we train an auto-encoder model to compress the representation into a compact latent space, which not only enables efficient training of complex sewing pattern generation with fixed memory consumption but also ensures scalability for future applications involving more intricate garment designs.
To achieve multi-modal controlled and body-aware sewing pattern generation, we design a two-step training strategy to introduce the control signals into the latent diffusion model.
In the first step, we train the diffusion model under the condition of texts, serving as a coarse fundamental model for additional control signal injection.
In the second step, we further embed the knowledge of sketches and body shapes into the diffusion model by fusing the features after the first block and fine-tuning the output layers within the attention module, providing additional control of garment details and ensuring the generated garments fit various body shapes end to end without the need of delicate body measurements.
Based on the proposed framework, SewingLDM can generate complex garments that fit various body shapes and align with user-provided text descriptions or garment sketches.

Our generated sewing patterns can be seamlessly integrated into subsequent CG pipelines, facilitating editing and animation processes.
After simulation, the garment mesh can be combined with current texture generation methods [73, 67, 70, 38] or handcrafted texture to generate colored garments, as shown in [Fig. 1](https://arxiv.org/html/2412.14453v2#S0.F1), demonstrating our fantastic generation ability.
Comprehensive qualitative and quantitative experiments show the superiority of our proposed method in terms of complex garment design and various body adaptability compared with previous methods.
To summarize, our main contributions include:

- •
We design a novel architecture, dubbed SewingLDM, for sewing pattern generation conditioned by texts, body shapes, and garment sketches, enabling precisely controlled and body-suited garment generation.
- •
We design an extended representation to cover complex sewing patterns and compress it into a compact latent space enabling the training of complex sewing patterns generation and maintaining high reconstruction quality.
- •
We design a two-step training strategy to better inject the multi-modal control signals into a diffusion model, yielding superior generation performance and controllability.

## 2 Related Work

Figure: Figure 2: Sewing pattern. Sewing patterns are CAD representations of garments, containing 2D shapes and 3D placement of cloth. They consist of panels, and panels consist of edges joined from beginning to end. Between panels or inner panels, stitches are used to connect edges to form clothes. For each edge, different kinds of lines are utilized to conform to body contours. Additionally, complex sewing patterns need additional attachment constraints during simulation for certain edges, which is highlighted in red in the attachment types region. Besides, for the stitch between panels and inner panels, a reversal stitch flag is sometimes needed to reverse the stitch direction.
Refer to caption: x1.png

Multimodal-guided 3D Generation.
The advancements in large models [52, 54, 1] have encouraged the emergence of recent 3D generation models [61, 24, 22, 43, 12, 35, 72, 58], entering a new era of 3D generation.
Some models [50, 59, 69, 9, 42, 39] focus on generating implicit neural radiance fields corresponding to the text description, whereas others [60, 12, 36] extend their ability to generate 3D meshes with BRDF materials.
Especially, some models [55, 56, 30, 39] dive into human clothes generation.
Garment3DGen [55] can generate textured 3D mesh from images or text conditions by adjusting template meshes.
GarmentDreamer [30] utilizes 3D Gaussian Splatting (GS) [25] as guidance to create 3D garment meshes from textual prompts.
WordRobe [56] leverages unsigned distance field (UDF) [19] to represent 3D garments and generate 3D garment meshes under text guidance.
GarVerseLOD [39] proposes a hierarchical framework to recover different levels of garment details through single images.
Although these methods can generate visual-appealing garment meshes, their compatibility with CG pipelines remains a challenge, hindering seamless integration into modern industry workflows.
In contrast, our method aims at sewing pattern generation, facilitating utilization in CG processes.

3D Sewing Pattern Modeling.
Existing fashion CAD software tools, such as Clo3D [15] and Marvelous Designer [16], allow users to edit sewing patterns and simulate desired cloth outcomes.
While these methods integrate the most advanced garment design, they heavily rely on artists to manually draw and adjust the shapes of sewing patterns, requiring a substantial of professional manual processing.
Consequently, ongoing studies are focusing on automating the adjustment of sewing patterns [6, 62, 40, 33, 64, 49, 17], reconstructing sewing patterns [23, 66, 20, 13, 5, 27, 34], and assisting with complex garment design [31, 63, 18, 14, 32].
Recent studies [34, 21, 27, 28, 7, 46, 76] on sewing patterns start to focus on autonomously generating diverse sewing patterns through different conditions rather than merely adjusting or producing a single garment.
One of the recent SOTA methods, DressCode [21], first generates garments through natural language and yields visual-appealing appearances.
However, the capability of DressCode is limited in modeling complex sewing patterns, and furthermore, it does not consider the relation between garments and body shapes, difficult to drape on various bodies.
Another typical work, parametric sewing pattern [28], can control complex sewing patterns, while it requires predefined templates and a delicate selection of different control scale values, which is not user-friendly.
Different from these methods, our SewingLDM not only has the ability to represent complex sewing patterns, but can also generate sewing patterns based on multi-modal intuitive conditions, *i.e*., natural language, garment sketches, and body shape.
These capabilities enable easily creating tailored garments that conform precisely to individual body shapes.

## 3 Method

To generate garments suited for various humans, we introduce SewingLDM, a latent-based diffusion model, to create complex 3D sewing patterns, conditioned by personalized body shapes, text prompts, and garment sketches.
We first review the original sewing pattern representation [27] ([Sec. 3.1](https://arxiv.org/html/2412.14453v2#S3.SS1)) and then we improve it with special designs to cover complex sewing patterns ([Sec. 3.2](https://arxiv.org/html/2412.14453v2#S3.SS2)), as illustrated in [Fig. 2](https://arxiv.org/html/2412.14453v2#S2.F2).
Subsequently, we compress the sewing pattern representation into a compact latent space enabling the generation of complex sewing patterns with centimeter-level precision in [Fig. 3](https://arxiv.org/html/2412.14453v2#S3.F3) ([Sec. 3.3](https://arxiv.org/html/2412.14453v2#S3.SS3)).
Finally, we train a latent diffusion model under multi-modal conditions through a two-stage training strategy, as illustrated in [Fig. 4](https://arxiv.org/html/2412.14453v2#S3.F4) ([Sec. 3.4](https://arxiv.org/html/2412.14453v2#S3.SS4)).
Based on the proposed framework, SewingLDM can generate complex garments based on the body shape and align with the user-provided text description or garment sketches.

Figure: Figure 3: Sewing pattern compression. We compress the sewing pattern representations into a bound and compact latent space.
Refer to caption: x2.png

Figure: Figure 4: Multimodal latent diffusion model. After training the text-guided diffusion model, we fuse the features of sketches and body shapes and normalize them into the diffusion model, with minimal fine-tuning parameters of the diffusion model. The trained network parameters are depicted in orange, while the frozen parameters are shown in purple. The output latent is then quantized into a designed latent space and serves as the input of the decoder to yield all edge lines. Edge lines connect from beginning to end to form panels, placed on the corresponding body regions. Finally, we can get suited garments through the modern CG pipeline.
Refer to caption: extracted/6600879/figs/pipeline.png

### 3.1 Preliminaries on Sewing Pattern

Sewing patterns are CAD representations of garments, representing 2D shapes and 3D placement of cloth, as shown in [Fig. 2](https://arxiv.org/html/2412.14453v2#S2.F2).
NeuralTailor [27] first transfers sewing patterns into vector representations as inputs of the neural network.
The sewing pattern contains $N_{p}$ panels $\{P_{i}\}_{i=1}^{N_{p}}$, with each panel $P_{i}$ including $N_{i}$ edges $\{E_{i,j}\}_{j=1}^{N_{i}}$.
For each edge $E_{i,j}$, a vector $\{V_{i,j}\}_{j=1}^{N_{i}}\in\mathbb{R}^{2}$ is utilized to represent the direction from its starting to ending point.
In NeuralTailor [27], sewing patterns only have two kinds of edges, *i.e*., straight lines and quadratic lines.
Quadratic lines use two additional parameters $C_{i,j}=(c_{x},c_{y})$ representing the control point of the Bezier curve.
Rotation $R_{i}\in SO(3)$ and translation $T_{i}\in\mathbb{R}^{3}$ are utilized to represent the 3D placement of each panel $P_{i}$.
Moreover, to depict the stitching connecting each inner or outer panel edge, it incorporates per-edge stitch tags $\{S_{i,j}\in\mathbb{R}^{3}\}_{j=1}^{N_{i}}$ and stitch masks $\{M_{i,j}\in\{0,1\}\}_{j=1}^{N_{i}}$.
The stitch tag $S_{i,j}$ is determined by the 3D position of the corresponding edge, which utilizes the Euclidean distance between edges as a measure of stitch similarity.
The stitch mask $M_{i,j}$ is a binary flag to indicate whether there are stitches on the edge.

### 3.2 Extended Representation for Sewing Pattern

For more complex garment designs, the modern industry will use special designs to make garments more fashionable, as illustrated in [Fig. 2](https://arxiv.org/html/2412.14453v2#S2.F2).
Original sewing pattern representation in [27] can not cover complex garments with more kinds of curve lines, *i.e*., cubic and circle lines, and additional attachment constraints in collars and waistbands to prevent cloth sliding from human bodies for certain garments, *e.g*., strapless tops, and loose pants.
Moreover, the stitches may intersect for some edge pairs, causing errors during simulation.
To precisely represent complex clothes, we extend each edge feature to high dimensions to cover complex patterns.
Then we preprocess them into a uniform tensor shape to feed into the neural network.

Representation.
For each edge $E_{i,j}$, we append origin control parameters $C_{i,j}$ with cubic line control parameters $C^{b}_{i,j}\in\mathbb{R}^{4}$, representing two control points, and circle line control parameters $C^{r}_{i,j}\in\mathbb{R}^{3}$, which represents the radius $r$ and the rotation angle, to cover more kinds of curve lines.
Further, we use two binary flags $\{E^{t}_{i,j,k}\in\{0,1\}\}_{k=1}^{2}$ to denote these 4 different edge types.
Moreover, 3 specific binary flags $\{A_{i,j,k}\in\{0,1\}\}_{k=1}^{3}$ are included to indicate the attachment type for certain edges, such as those associated with the collar and waistband, to prevent garments from sliding during simulation.
We add one binary flag to ${M_{i,j}}$ as the new stitch mask $\{M_{i,j,k}^{{}^{\prime}}\in\{0,1\}\}_{k=1}^{2}$ to additionally denote whether the stitch direction needs to be reversed to prevent stitch intersections.

Preprocessing.
Before input to the neural networks, the vector representation needs to be the same size for all data during training.
For each edge $E_{i,j}$, we concatenate all extended parameters and append with rotation $R_{i}$ and translation $T_{i}$ of panel $P_{i}$ to form the high-dimensional edge feature $E^{f}_{i,j}$.
Furthermore, we design a binary flag $\{E^{m}_{i,j}\in\{0,1\}\}_{j=1}^{N_{i}}$ to denote the existence of each edge.
All features are concatenated to form a 29-dimensional vector for each edge feature $E^{f}_{i,j}$, represented as follows:

$$ $\displaystyle E^{f}_{i,j}=V_{i,j}$ $\displaystyle\oplus C_{i,j}\oplus C^{b}_{i,j}\oplus C^{r}_{i,j}\oplus S_{i,j} \oplus R_{i}$ (1) $\displaystyle\oplus T_{i}\oplus E^{t}_{i,j}\oplus E^{m}_{i,j}\oplus A_{i,j} \oplus M_{i,j}^{{}^{\prime}},$ $$

where $i$ is in the range of $[1,max(N_{p})]$, $j$ is in the range of $[1,max(N_{i})]$.

Then, all edge features $\{E^{f}_{i,j}\}_{j=1}^{N_{i}}$ are concatenated and padded with 0 to max edge number and max panel number to get the representation of sewing pattern $F$, in the shape of $(max(N_{p})\times max(N_{i}),29)$.
Before input to the neural network, all continuous values are standardized, and all binary flags are transformed into $\{-1,1\}$.

### 3.3 Compact Latent for Sewing Pattern

The vector representation $F$ inevitably incorporates redundant information as the panel and edge numbers increase, preventing generation models from learning the distribution of $F$.
Using the tokenization in the previous method [21] will lead to excessive GPU consumption, exceeding hardware limitations.
As indicated by the recent compression methods [74, 41, 68], it is necessary to compress $F$ into a compact latent space and maintain the reconstruction quality.
Following this idea, we train an auto-encoder to compress and quantize the $F$ into a latent space where each dimension is bounded within the range $\left[-1,1\right]$, as illustrated in [Fig. 3](https://arxiv.org/html/2412.14453v2#S3.F3).
The sewing pattern representation $F$ is encoded to $z$ by the encoder $\mathcal{E}$ and quantized to $\hat{z}$ in the constrained latent space, subsequently reconstructed by the decoder $\mathcal{D}$. The process can be represented as:

$$ $z=\mathcal{E}(F_{\mathrm{gt}}),\\ \hat{z}=\frac{round(n\times tanh(z))}{n},\\ F_{\mathrm{rec}}=\mathcal{D}(\hat{z}),$ (2) $$

where $n$ is an integer used to modify the spacing between each $\hat{z}$ in the latent space.
For training the encoder $\mathcal{E}$ and decoder $\mathcal{D}$, we combine the loss in previous works [27, 41] with additional binary cross-entropy loss $\mathcal{L}_{\mathrm{BCE}}$ to constrain the newly incorporated binary flags:

$$ $\mathcal{L}_{\mathrm{total}}=\lambda_{1}\mathcal{L}_{\mathrm{rec}}+\lambda_{2} \mathcal{L}_{\mathrm{panel}}+\lambda_{3}\mathcal{L}_{\mathrm{stitch}}+\lambda_ {4}\mathcal{L}_{\mathrm{BCE}},$ (3) $$

where $\lambda_{1},\lambda_{2},\lambda_{3},\lambda_{4}$ are hyperparameters to balance each loss term.
$\mathcal{L}_{\mathrm{rec}}$ is the MSE loss to keep reconstruction quality, while $\mathcal{L}_{\mathrm{panel}}$ and $\mathcal{L}_{\mathrm{stitch}}$ proposed in  [27] to ensure the integrity of the garments.

After training, the sewing pattern representation $F$ can be efficiently compressed into a bounded and compact latent space without compromising important information.
Moreover, to facilitate the learning of generation models, each dimension of the latent is evenly distributed within the coordinates $\{-1,-0.5,0,0.5,1\}$ by setting $n=2$ in [Eq. 2](https://arxiv.org/html/2412.14453v2#S3.E2).

### 3.4 Multimodal Conditions of Diffusion Model

Inspired by the great power of controlled generation in the diffusion model, we employ latent diffusion [54] as our generation model.
Our generation model is based on the DiT architecture [48, 11], which is scalable to different sizes of sewing patterns.
To balance multi-modal conditions and facilitate future conditional scalability, we design a two-step training strategy: 1) In the first step, we train the latent diffusion model with IDDPM loss [47] only under the text guidance extracted by T5 tokenizer [53]; 2) In the second step, we embed the knowledge of body shapes and garment sketches into the diffusion model for detailed control and body-suited garment generation, as depicted in [Fig. 4](https://arxiv.org/html/2412.14453v2#S3.F4).

The text-guided latent diffusion model serves as a fundamental model for extensive multi-modal conditions injection.
For the injection of body shapes and garment sketches, a naive idea is to inject them through two ControlNet [71] branches.
While sketches will change along with body shape, *e.g*., sketches will get wider when bodies grow fatter.
We propose to first use embedders to extract the feature from sketches and body shapes, and then simply concatenate them together, and input them into a light transformer layer.
During the light transformer layer, the features of sketches and body shapes can be thoroughly fused and get the relation between each other through self-attention modules and output as $\bm{F}_{bs}$.
Then we normalize mean $\bm{\mu}_{bs}$ and variance $\bm{\sigma}_{bs}$ of $\bm{F}_{bs}$ into the same mean $\bm{\mu}_{z}$ and variance $\bm{\sigma}_{z}$ with the latent features $\bm{F}_{z}$, serving as a residual to control the output.
The new $\hat{\bm{F}}_{z}$ is represented as below:

$$ $\hat{\bm{F}}_{z}=\frac{(\bm{F}_{bs}-\bm{\mu}_{bs})\times\bm{\sigma}_{bs}}{\bm{ \sigma}_{z}+\epsilon}+\bm{\mu}_{z}+\bm{F}_{z},$ (4) $$

where $\epsilon$ is a small constant for numerical stability.
After the normalization, we assume the newly added residual is similar to $\bm{F}_{z}$, which does not need to retrain the whole diffusion model in the second stage.
We only fine-tune the output layer of the attention modules in each DiT block to transform the normalized features into the desired distribution.
After two-stage training, our generation model can precisely follow the text guidance and sketches under various human shapes, enabling more body-suited and detailed controlled garment generation for individuals.

## 4 Experiments

### 4.1 Experiment Setup

Dataset. To train a generation model under the condition of texts or sketches, it is essential to acquire the corresponding paired data.
We extend the current dataset [29] with additional textual annotations and garment sketches.
The dataset [29] consists of 120,000 sewing patterns, covering a variety of clothing styles for different body types.
Sewing patterns in [29] consider the relationship between various body shapes and garments, resulting in garments that are well-tailored to individual body types.
Building on [29], we annotate each garment with text prompts according to its design parameters file and refine it with GPT-4 [1], resulting in detailed text annotations.
However, relying solely on textual descriptions may not precisely dictate garment shapes, potentially yielding undesirable outputs.
To enhance control over the generation, we propose to generate richer annotations like sketches.
For each garment, we utilize PiDiNet [57], a pre-trained edge detection network, to extract garment sketches, thereby enriching the design details of the garment.
More annotation details are represented in the supplementary material.

Implementation Details. We train our model on 4 RTX A6000 GPUs with 48G memory, where the auto-encoder requires 12 hours for training.
The hyperparameters for training the auto-encoder, $\lambda_{1},\lambda_{2},\lambda_{3},\lambda_{4}$ are set as $5,1,1,1$.
Training the text-guided latent diffusion model takes 2 days in the first stage, and training the multi-modal conditions requires an additional 10 hours to reach convergence in the second stage.
During the second stage, the sketch or text conditions are set to zero with a probability of 0.25 to ensure the model retains the capability to generate desired garments based on a single input condition.

Figure: Figure 5: Comparison with 3D mesh generation method. We present the garments and draping results for each method. Our method successfully generates modern design garments with remarkable visual quality and close fitting to various body shapes. In contrast, Wonder3D [37] and RichDreamer [51] only generate close-surface meshes and contain obvious artifacts, resulting in human bodies clipping through the garments.
Refer to caption: x3.png

### 4.2 Qualitative Comparison

We conduct qualitative comparisons with SOTA mesh generation methods and sewing pattern generation methods, respectively, to demonstrate our CG-friendly and superior generation results for various body shapes.

Figure: Figure 6: Comparison of sewing pattern generation for various body shapes. For each method, we present corresponding conditions and generated results, including sewing patterns, draping results on average body shape, and draping results on another body shape. Our method can generate complicated sewing patterns aligned with sketch conditions and text prompts, draping on various body shapes. In contrast, Sewformer [34] and DressCode [21] both fail to generate complex and body-suited garments.
Refer to caption: x4.png

Comparison on 3D Mesh Generation Methods. We compare our SewingLDM with the current SOTA 3D mesh generation methods, *i.e*., Wonder3D [37] and RichDreamer [51] with the same text prompts containing both the geometry and texture information.
Note that, RichDreamer [51] is an image-guided generation method; thus, we feed the sketches and text prompts to ControlNet [71] with Stable Diffusion [54] to generate corresponding images.
As the qualitative comparisons shown in [Fig. 5](https://arxiv.org/html/2412.14453v2#S4.F5), Wonder3D and RichDreamer can both generate garments aligned with text prompts.
However, both of them are close-surface meshes and do not consider human body shapes, resulting in obvious clipping when draping on human bodies.
In contrast, our method generates sewing patterns for various human bodies through two-stage training, which are easy to drape on human bodies and maintain the physical properties of clothing with fantastic clothes wrinkles.
The results show that our garments are all well-fitted with complex geometry and aligned with the conditions.

Comparison of Sewing Pattern Generation Methods. We also compare our method with current SOTA sewing pattern generation methods, *i.e*., DressCode [21] and Sewformer [34].
To ensure a fair comparison, we directly utilize the in-the-wild images chosen from the Multimodal Garment Designer [4] and transfer into the corresponding conditions of DressCode and Sewformer.
Moreover, since Sewformer and DressCode are only trained with the average SMPL body shapes, we drape the generative sewing patterns onto the average body shape and another body shape to validate the effectiveness of body-aware garment generation.
As illustrated in [Fig. 6](https://arxiv.org/html/2412.14453v2#S4.F6), both Sewformer and DressCode fail to generate textual-aligned garments due to the complex garment descriptions.
Moreover, since these baseline methods do not account for various bodies and generate attachments to any edge of sewing patterns, they can not be worn on diverse body shapes, only on the average body shape, sliding from the body or just failing to simulate the results.
In contrast, our method uses a compact latent space to represent complex sewing patterns with centimeter-level precision, which greatly improves the generation ability on complex sewing pattern generation, *e.g*., fitted shirts, one-shoulder gowns, and square necklines.
Moreover, due to the body shape conditions, our method can fit various body shapes, which provides a more user-friendly approach to getting the desired made-to-measure garments.
The results demonstrate the superiority of our method in complex garment design and various body adaptability.
We also provide concurrent work comparison and more in-the-wild and in-domain conditions in the supplementary material.

**Table 1: Quantitative Comparison. We compare the generation efficiency, the average clothes-to-body distance, and users’ evaluation. All metrics show that our method generated superior results.**
|  | RichDreamer | Wonder3D | Sewformer | Dresscode | Ours |
| --- | --- | --- | --- | --- | --- |
| Runtime $\downarrow$ | $\sim$ 4 mins | $\sim$ 4 hours | $\sim$ 3 mins | $\sim$ 3 mins | $\sim$ 3 mins |
| Clothes-to-body<br>Distance<br>$\downarrow$ | 6.19 cm | 6.54 cm | 5.45 cm | 3.69 cm | 2.20 cm |
| Users Values $\uparrow$ | 1.89 | 1.88 | 2.10 | 3.56 | 4.60 |

**Table 2: Reconstruction metrics. We report the reconstruction performance on the test set of the GarmentCode dataset [29]. Our auto-encoder (AE) can reconstruct the sewing pattern with centimeter-level precision compliant with industrial standards.**
| Method | Panel L2$(\downarrow)$ | Panel Acc$(\uparrow)$ | Edge Acc$(\uparrow)$ | Rot L2$(\downarrow)$ | Transl L2$(\downarrow)$ | Stitch Acc$(\uparrow)$ | Failure Rate$(\downarrow)$ |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SewFormer* | 12.3 | 79.4 | 44.7 | .0400 | 4.5 | 2.8 | 4.3% |
| AE (Ours) | 0.64 | 99.8 | 88.5 | .0004 | 0.6 | 90.8 | 0 |
| SewingLDM | 3.13 | 97.8 | 82.7 | .0043 | 1.2 | 84.2 | 0 |

### 4.3 Quantitative Comparison

Besides qualitative comparisons, we also perform quantitative comparisons with these SOTA methods, evaluating aspects including generation efficiency, clothes-to-body distance, and user study.
The clothes-to-body distance is an essential metric for body-aware garment generation, indicating whether the clothes are close-fitted to body shapes calculated by averaging the minimum distance from each garment point to human bodies.
Besides, we also perform a user study to further assess the quality of garment generation.
We take 10 text prompts to generate diverse garments and render the generated results, draping on different body shapes for each method.
Then we ask 30 users to give a value of these rendering results with comprehensive consideration for two aspects: 1) consistency with text descriptions; and 2) well-fitting with human bodies.
As illustrated in [Tab. 1](https://arxiv.org/html/2412.14453v2#S4.T1), the preference results demonstrate a notable superiority of our method over SOTA approaches in both aspects, highlighting in generating garments that are both well-suited to various bodies and exhibit high fidelity aligned with text descriptions.
Furthermore, we also evaluate sewing pattern reconstruction metrics as proposed in SewFormer [34], including: 1) Panel L2, Rot L2, and Transl L2 (average edge, rotation, and translation L2 distances between predicted and ground-truth patterns); 2) Panel Accuracy, Edge Accuracy, Stitch Accuracy (accuracy of panel count, edge prediction per panel, stitch prediction); and 3) Failure Rate (simulation failure percentage).
All L2 metrics are in centimeters, except rotation in radians.
To ensure a fair evaluation, SewFormer* is fine-tuned on the GarmentCode dataset until its validation loss no longer improves.
Dresscode, which requires over 30k tokens for complex patterns and exceeds current GPU limitations, unabling to train on the GarmentCode dataset, was evaluated only for failure rate (11.4%) using its open-source code and checkpoint.
As shown in [Tab. 2](https://arxiv.org/html/2412.14453v2#S4.T2), our method achieved superior performance compared with baselines on complex sewing pattern generation as our latent space is well-designed and suitable for centimeter-level garments compression and our model takes the body shapes into account.

### 4.4 Ablation Study

Figure: Figure 7: Ablation of the multi-modal condition. We have taken an ablation experiment on the training parameters of the output layer in different attention modules. We also explore the relationship between results and injection positions.
Refer to caption: x5.png

Sewing Pattern Compression. To maintain the reconstruction ability and dense compression of sewing patterns in the meantime, we have explored numerous parameters for the compression network, as shown in [Tab. 3](https://arxiv.org/html/2412.14453v2#S4.T3).
We measure the reconstruction ability, generation ability, clothes-to-body distance, and codebook usage under different settings.
The codebook usage is calculated by the number of used latent $N_{U}$ dividing the latent number in latent space $N_{L}$ as follows:

$$ $codebook~{}usage=\frac{N_{U}}{N_{L}}=\frac{N_{U}}{{(2n+1)}^{n_{f}}},$ (5) $$

where $n$ is an integer in pre-defined [Eq. 2](https://arxiv.org/html/2412.14453v2#S3.E2) for quantization, $n_{f}$ is the last dimension length of latent.
As illustrated in [Tab. 3](https://arxiv.org/html/2412.14453v2#S4.T3), without compression or lower compression, the latent space is inappropriate for the generation model to learn the distribution of latent.
In contrast, with a compact latent space, the latent is fully utilized, resulting in a well-generation ability and various body adaptability.

**Table 3: Different Compression. We try different compression shapes for the latent and set the different values of n. ✓means it can well do this task, while $\times$ means it fails in this task.**
| Compression shape | w/o<br>compression | 256*32<br>n=32 | 256*12<br>n=8 | 256*8<br>n=2 | 256*6<br>n=2 | 256*4<br>n=2 |
| --- | --- | --- | --- | --- | --- | --- |
| Reconstruction | ✓ | ✓ | ✓ | ✓ | ✓ | $\times$ |
| Generation | $\times$ | $\times$ | $\times$ | ✓ | ✓ | - |
| Clothes-to-body<br>Distance<br>$\downarrow$ | - | - | - | 2.87 cm | 2.20 cm | - |
| Codebook usage | - | 0% | 0% | 91% | 100% | - |

Multi-modal Controllable Generation. In the context of injecting body shape and garment sketch conditions, we perform ablation studies on the optimized parameters of the output layers across different attention modules, *i.e*., both self-attention and cross-attention, self-attention only, and cross-attention only.
By default, we inject the additional condition after the first transformer block.
Moreover, we investigate the impact of different injection positions, specifically after block 5, block 10, block 15, and block 20, as illustrated in  [Fig. 7](https://arxiv.org/html/2412.14453v2#S4.F7).
Notably, optimizing in both attention results in more desired circle necklines than only optimizing in cross-attention and is better aligned with sketches compared with only training output layers in self-attention, which fails to generate the desired neckline and sleeves.
This shows that the additional conditional can closely resemble the latent features, which needs more learning across text prompts and latents rather than within latents alone.
Consequently, during the ablation study on injection positions, we fine-tune the output layers of both attention for better results.
We observe that as the layer depth increases, the garment gradually loses key components, *e.g*., sleeves or waistband, resulting in unable draping on the human body.
In contrast, injecting the condition at shallower layers facilitates better fusion of the additional condition with the combined latent feature, leading to more accurate results.
In summary, we train the output layers of both attention and inject the additional condition after block 0, which yields optimal results.

## 5 Conclusion and Limitations

Figure: Figure 8: Limitations. For intricate sketches, such as bridal gowns or additional accessories like pockets and zippers, our method may fail to generate the desired garments.
Refer to caption: x6.png

In conclusion, our SewingLDM can generate complex sewing patterns under the condition of text prompts, garment sketches, and body shapes.
We propose an enhanced vector representation of sewing patterns and compress them into a bounded and compact latent space for more generalized garment designs and facilitating the training of the diffusion model.
To accommodate multi-modal conditioning and future conditions, we introduce a two-step training strategy.
We first train a latent diffusion model only conditioned by text prompts.
Subsequently, we incorporate the condition of garment sketches and body shapes by optimizing the output layers of the attention modules while maintaining the responsiveness to text-based guidance.
Finally, our generation model can be conditioned by multi-modal input, resulting in body-suited generation and detailed control of garments.

Despite the promising results, our method still has several limitations that should be addressed in future work, as illustrated in [Fig. 8](https://arxiv.org/html/2412.14453v2#S5.F8).
The major limitation is that our method encounters challenges with certain modern designs, *e.g*., zippers, and pockets.
Another limitation is that it occasionally struggles with aligning complex sketches of intricate garments.
Our further work aims to explore comprehensive representations of daily garments and expand the range of conditions applicable during the generation process.

## References

- Achiam et al. [2023]
Josh Achiam, Steven Adler, Sandhini Agarwal, Lama Ahmad, Ilge Akkaya,
Florencia Leoni Aleman, Diogo Almeida, Janko Altenschmidt, Sam Altman,
Shyamal Anadkat, et al.
Gpt-4 technical report.
*arXiv preprint arXiv:2303.08774*, 2023.
- Adobe [2024]
Adobe.
Substance 3D Painter.
https://creativecloud.adobe.com/apps/all/substance3d-painter, 2024.
- Autodesk, INC. [2019]
Autodesk, INC.
Maya.
https://autodesk.com/maya, 2019.
- Baldrati et al. [2023]
Alberto Baldrati, Davide Morelli, Giuseppe Cartella, Marcella Cornia, Marco
Bertini, and Rita Cucchiara.
Multimodal garment designer: Human-centric latent diffusion models
for fashion image editing.
In *ICCV*, pages 23393–23402, 2023.
- Bang et al. [2021]
Seungbae Bang, Maria Korosteleva, and Sung-Hee Lee.
Estimating garment patterns from static scan data.
*Computer Graphics Forum*, 40(6):273–287,
2021.
- Bartle et al. [2016]
Aric Bartle, Alla Sheffer, Vladimir G. Kim, Danny M. Kaufman, Nicholas Vining,
and Floraine Berthouzoz.
Physics-driven pattern adjustment for direct 3D garment editing.
*TOG*, 35(4):50–1, 2016.
- Bian et al. [2025]
Siyuan Bian, Chenghao Xu, Yuliang Xiu, Artur Grigorev, Zhen Liu, Cewu Lu,
Michael J Black, and Yao Feng.
Chatgarment: Garment estimation, generation and editing via large
language models.
In *CVPR*, 2025.
- Blender Foundation [2022]
Blender Foundation.
Blender.
https://www.blender.org/, 2022.
- Cao et al. [2024]
Yukang Cao, Yan-Pei Cao, Kai Han, Ying Shan, and Kwan-Yee K Wong.
Dreamavatar: Text-and-shape guided 3d human avatar generation via
diffusion models.
In *CVPR*, pages 958–968, 2024.
- Chen et al. [2024a]
Beijia Chen, Yuefan Shen, Qing Shuai, Xiaowei Zhou, Kun Zhou, and Youyi Zheng.
Anidress: Animatable loose-dressed avatar from sparse views using
garment rigging model.
*arXiv preprint arXiv:2401.15348*, 2024a.
- Chen et al. [2024b]
Junsong Chen, Jincheng Yu, Chongjian Ge, Lewei Yao, Enze Xie, Zhongdao Wang,
James T. Kwok, Ping Luo, Huchuan Lu, and Zhenguo Li.
Pixart-$\alpha$: Fast training of diffusion transformer for
photorealistic text-to-image synthesis.
In *ICLR*, 2024b.
- Chen et al. [2023]
Rui Chen, Yongwei Chen, Ningxin Jiao, and Kui Jia.
Fantasia3d: Disentangling geometry and appearance for high-quality
text-to-3d content creation.
In *ICCV*, pages 22246–22256, 2023.
- Chen et al. [2015]
Xiaowu Chen, Bin Zhou, Feixiang Lu, Lin Wang, Lang Bi, and Ping Tan.
Garment modeling with a depth camera.
*TOG*, 34(6):1–12, 2015.
- Chowdhury et al. [2022]
Pinaki Nath Chowdhury, Tuanfeng Wang, Duygu Ceylan, Yi-Zhe Song, and Yulia
Gryaditskaya.
Garment Ideation: Iterative View-Aware Sketch-Based Garment
Modeling.
In *International Conference on 3D Vision*, 2022.
- CLO Virtual Fashion [2022]
CLO Virtual Fashion.
Clo3d.
https://clo3d.com/en/, 2022.
- CLO Virtual Fashion [2024]
CLO Virtual Fashion.
Marvelous Designer.
https://www.marvelousdesigner.com/, 2024.
- Feng et al. [2024]
Xudong Feng, Huamin Wang, Yin Yang, and Weiwei Xu.
Neural-assisted homogenization of yarn-level cloth.
In *SIGGRAPH*, pages 1–10, 2024.
- Fondevilla et al. [2021]
Amelie Fondevilla, Damien Rohmer, Stefanie Hahmann, Adrien Bousseau, and
Marie Paule Cani.
Fashion Transfer: Dressing 3D Characters from Stylized Fashion
Sketches.
*Computer Graphics Forum*, 40(6):466–483,
2021.
- Guillard et al. [2022]
Benoit Guillard, Federico Stella, and Pascal Fua.
Meshudf: Fast and differentiable meshing of unsigned distance field
networks.
In *ECCV*, 2022.
- Hasler et al. [2007]
Nils Hasler, Bodo Rosenhahn, and Hans Peter Seidel.
Reverse engineering garments.
In *International Conference on Computer Vision/Computer
Graphics Collaboration Techniques and Applications*, pages 200–211, 2007.
- He et al. [2024]
Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, and Lan Xu.
Dresscode: Autoregressively sewing and generating garments from text
guidance.
*TOG*, 43(4):1–13, 2024.
- Jain et al. [2022]
Ajay Jain, Ben Mildenhall, Jonathan T Barron, Pieter Abbeel, and Ben Poole.
Zero-shot text-guided object generation with dream fields.
In *CVPR*, pages 867–876, 2022.
- Jeong et al. [2015]
Moon-Hwan Jeong, Dong-Hoon Han, and Hyeong-Seok Ko.
Garment capture from a photograph.
*Computer Animation and Virtual Worlds*, 26(3-4):291–300, 2015.
- Jetchev [2021]
Nikolay Jetchev.
Clipmatrix: Text-controlled creation of 3d textured meshes.
*arXiv preprint arXiv:2109.12922*, 2021.
- Kerbl et al. [2023]
Bernhard Kerbl, Georgios Kopanas, Thomas Leimkühler, and George Drettakis.
3d gaussian splatting for real-time radiance field rendering.
*TOG*, 42(4):139–1, 2023.
- Kim et al. [2024]
Jeongho Kim, Guojung Gu, Minho Park, Sunghyun Park, and Jaegul Choo.
Stableviton: Learning semantic correspondence with latent diffusion
model for virtual try-on.
In *CVPR*, pages 8176–8185, 2024.
- Korosteleva and Lee [2022]
Maria Korosteleva and Sung-Hee Lee.
Neuraltailor: Reconstructing sewing pattern structures from 3d point
clouds of garments.
*TOG*, 41(4):1–16, 2022.
- Korosteleva and Sorkine-Hornung [2023]
Maria Korosteleva and Olga Sorkine-Hornung.
Garmentcode: Programming parametric sewing patterns.
*TOG*, 42(6):1–15, 2023.
- Korosteleva et al. [2024]
Maria Korosteleva, Timur Levent Kesdogan, Fabian Kemper, Stephan Wenninger,
Jasmin Koller, Yuhan Zhang, Mario Botsch, and Olga Sorkine-Hornung.
Garmentcodedata: A dataset of 3d made-to-measure garments with sewing
patterns.
In *ECCV*, 2024.
- Li et al. [2024]
Boqian Li, Xuan Li, Ying Jiang, Tianyi Xie, Feng Gao, Huamin Wang, Yin Yang,
and Chenfanfu Jiang.
Garmentdreamer: 3dgs guided garment synthesis with diverse geometry
and texture details.
*arXiv preprint arXiv:2405.12420*, 2024.
- Li et al. [2018]
Minchen Li, Alla Sheffer, Eitan Grinspun, and Nicholas Vining.
FoldSketch: Enriching garments with physically reproducible folds.
*TOG*, 37(4):1–13, 2018.
- Liu et al. [2024a]
Chen Liu, Weiwei Xu, Yin Yang, and Huamin Wang.
Automatic digital garment initialization from sewing patterns.
*TOG*, 43(4):1–12, 2024a.
- Liu et al. [2018]
Kaixuan Liu, Xianyi Zeng, Pascal Bruniaux, Xuyuan Tao, Xiaofeng Yao, Victoria
Li, and Jianping Wang.
3D interactive garment pattern-making technology.
*CAD Computer Aided Design*, 104:113–124, 2018.
- Liu et al. [2023]
Lijuan Liu, Xiangyu Xu, Zhijie Lin, Jiabin Liang, and Shuicheng Yan.
Towards garment sewing pattern reconstruction from a single image.
*TOG*, 42(6):1–15, 2023.
- Liu et al. [2024b]
Minghua Liu, Chao Xu, Haian Jin, Linghao Chen, Mukund Varma T, Zexiang Xu, and
Hao Su.
One-2-3-45: Any single image to 3d mesh in 45 seconds without
per-shape optimization.
In *NeurIPS*, 2024b.
- Liu et al. [2024c]
Shengqi Liu, Zhuo Chen, Jingnan Gao, Yichao Yan, Wenhan Zhu, Jiangjing Lyu, and
Xiaokang Yang.
Directional texture editing for 3d models.
*Computer Graphics Forum*, 43(6):e15196,
2024c.
- Long et al. [2024]
Xiaoxiao Long, Yuan-Chen Guo, Cheng Lin, Yuan Liu, Zhiyang Dou, Lingjie Liu,
Yuexin Ma, Song-Hai Zhang, Marc Habermann, Christian Theobalt, et al.
Wonder3d: Single image to 3d using cross-domain diffusion.
In *CVPR*, pages 9970–9980, 2024.
- Lopes et al. [2024]
Ivan Lopes, Fabio Pizzati, and Raoul de Charette.
Material palette: Extraction of materials from a single image.
In *CVPR*, 2024.
- Luo et al. [2024]
Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Yinyu Nie,
Weikai Chen, and Xiaoguang Han.
Garverselod: High-fidelity 3d garment reconstruction from a single
in-the-wild image using a dataset with levels of details.
*TOG*, 2024.
- Meng et al. [2012]
Yuwei Meng, Charlie C.L. Wang, and Xiaogang Jin.
Flexible shape control for automatic resizing of apparel products.
*CAD Computer Aided Design*, 44(1):68–76,
2012.
- Mentzer et al. [2024]
Fabian Mentzer, David Minnen, Eirikur Agustsson, and Michael Tschannen.
Finite scalar quantization: VQ-VAE made simple.
In *ICLR*, 2024.
- Metzer et al. [2023]
Gal Metzer, Elad Richardson, Or Patashnik, Raja Giryes, and Daniel Cohen-Or.
Latent-nerf for shape-guided generation of 3d shapes and textures.
In *CVPR*, pages 12663–12673, 2023.
- Michel et al. [2022]
Oscar Michel, Roi Bar-On, Richard Liu, Sagie Benaim, and Rana Hanocka.
Text2mesh: Text-driven neural stylization for meshes.
In *CVPR*, pages 13492–13502, 2022.
- Morelli et al. [2022]
Davide Morelli, Matteo Fincato, Marcella Cornia, Federico Landi, Fabio Cesari,
and Rita Cucchiara.
Dress Code: High-Resolution Multi-Category Virtual Try-On.
In *ECCV*, 2022.
- Morelli et al. [2023]
Davide Morelli, Alberto Baldrati, Giuseppe Cartella, Marcella Cornia, Marco
Bertini, and Rita Cucchiara.
LaDI-VTON: Latent Diffusion Textual-Inversion Enhanced Virtual
Try-On.
In *ACMMM*, 2023.
- Nakayama et al. [2025]
Kiyohiro Nakayama, Jan Ackermann, Timur Levent Kesdogan, Yang Zheng, Maria
Korosteleva, Olga Sorkine-Hornung, Leonidas Guibas, Guandao Yang, and Gordon
Wetzstein.
Aipparel: A large multimodal generative model for digital garments.
In *CVPR*, 2025.
- Nichol and Dhariwal [2021]
Alexander Quinn Nichol and Prafulla Dhariwal.
Improved denoising diffusion probabilistic models.
In *ICML*, pages 8162–8171, 2021.
- Peebles and Xie [2023]
William Peebles and Saining Xie.
Scalable diffusion models with transformers.
In *CVPR*, pages 4195–4205, 2023.
- Pietroni et al. [2022]
Nico Pietroni, Corentin Dumery, Raphael Guenot-Falque, Mark Liu, Teresa
Vidal-Calleja, and Olga Sorkine-Hornung.
Computational pattern making from 3D garment models.
*TOG*, 41(4), 2022.
- Poole et al. [2023]
Ben Poole, Ajay Jain, Jonathan T. Barron, and Ben Mildenhall.
Dreamfusion: Text-to-3d using 2d diffusion.
In *ICLR*, 2023.
- Qiu et al. [2024]
Lingteng Qiu, Guanying Chen, Xiaodong Gu, Qi Zuo, Mutian Xu, Yushuang Wu,
Weihao Yuan, Zilong Dong, Liefeng Bo, and Xiaoguang Han.
Richdreamer: A generalizable normal-depth diffusion model for detail
richness in text-to-3d.
In *CVPR*, pages 9914–9925, 2024.
- Radford et al. [2021]
Alec Radford, Jong Wook Kim, Chris Hallacy, Aditya Ramesh, Gabriel Goh,
Sandhini Agarwal, Girish Sastry, Amanda Askell, Pamela Mishkin, Jack Clark,
et al.
Learning transferable visual models from natural language
supervision.
In *ICML*, pages 8748–8763, 2021.
- Raffel et al. [2020]
Colin Raffel, Noam Shazeer, Adam Roberts, Katherine Lee, Sharan Narang, Michael
Matena, Yanqi Zhou, Wei Li, and Peter J Liu.
Exploring the limits of transfer learning with a unified text-to-text
transformer.
*JMLR*, 21(140):1–67, 2020.
- Rombach et al. [2022]
Robin Rombach, Andreas Blattmann, Dominik Lorenz, Patrick Esser, and Björn
Ommer.
High-resolution image synthesis with latent diffusion models.
In *CVPR*, pages 10684–10695, 2022.
- Sarafianos et al. [2024]
Nikolaos Sarafianos, Tuur Stuyck, Xiaoyu Xiang, Yilei Li, Jovan Popovic, and
Rakesh Ranjan.
Garment3dgen: 3d garment stylization and texture generation.
*arXiv preprint arXiv:2403.18816*, 2024.
- Srivastava et al. [2024]
Astitva Srivastava, Pranav Manu, Amit Raj, Varun Jampani, and Avinash Sharma.
Wordrobe: Text-guided generation of textured 3d garments.
In *ECCV*, pages 458–475, 2024.
- Su et al. [2023]
Zhuo Su, Jiehua Zhang, Longguang Wang, Hua Zhang, Zhen Liu, Matti Pietikäinen,
and Li Liu.
Lightweight pixel difference networks for efficient visual
representation learning.
*TPAMI*, 45(12):14956–14974, 2023.
- Sun et al. [2024]
Jia-Mu Sun, Tong Wu, and Lin Gao.
Recent advances in implicit representation-based 3d shape generation.
*Visual Intelligence*, 2(1):9, 2024.
- Tang et al. [2023]
Junshu Tang, Tengfei Wang, Bo Zhang, Ting Zhang, Ran Yi, Lizhuang Ma, and Dong
Chen.
Make-it-3d: High-fidelity 3d creation from a single image with
diffusion prior.
In *ICCV*, pages 22819–22829, 2023.
- Tang et al. [2024]
Jiaxiang Tang, Jiawei Ren, Hang Zhou, Ziwei Liu, and Gang Zeng.
Dreamgaussian: Generative gaussian splatting for efficient 3d content
creation.
In *ICLR*, 2024.
- Wang et al. [2022]
Can Wang, Menglei Chai, Mingming He, Dongdong Chen, and Jing Liao.
Clip-nerf: Text-and-image driven manipulation of neural radiance
fields.
In *CVPR*, pages 3835–3844, 2022.
- Wang et al. [2009]
Jin Wang, Guodong Lu, Weilong Li, Long Chen, and Yoshiyuki Sakaguti.
Interactive 3D garment design with constrained contour curves and
style curves.
*CAD Computer Aided Design*, 41(9):614–625,
2009.
- Wang et al. [2018]
Tuanfeng Y. Wang, Duygu Ceylan, Jovan Popović, and Niloy J. Mitra.
Learning a shared shape space for multimodal garment design.
*TOG*, 37(6):1–13, 2018.
- Wolff et al. [2023]
Katja Wolff, Philipp Herholz, Verena Ziegler, Frauke Link, Nico Brügel,
and Olga Sorkine-Hornung.
Designing Personalized Garments with Body Movement.
*Computer Graphics Forum*, 2023.
- Xiang et al. [2023]
Donglai Xiang, Fabian Prada, Zhe Cao, Kaiwen Guo, Chenglei Wu, Jessica Hodgins,
and Timur Bagautdinov.
Drivable avatar clothing: Faithful full-body telepresence with
dynamic clothing driven by sparse rgb-d input.
In *SIGGRAPH Asia*, pages 1–11, 2023.
- Yang et al. [2018]
Shan Yang, Zherong Pan, Tanya Amert, Ke Wang, Licheng Yu, Tamara Berg, and
Ming C. Lin.
Physics-inspired garment recovery from a single-view image.
*TOG*, 37(5):1–14, 2018.
- Youwang et al. [2024]
Kim Youwang, Tae-Hyun Oh, and Gerard Pons-Moll.
Paint-it: Text-to-texture synthesis via deep convolutional texture
map optimization and physically-based rendering.
In *CVPR*, pages 4347–4356, 2024.
- Yu et al. [2024]
Lijun Yu, José Lezama, Nitesh Bharadwaj Gundavarapu, Luca Versari, Kihyuk
Sohn, David Minnen, Yong Cheng, Agrim Gupta, Xiuye Gu, Alexander G.
Hauptmann, Boqing Gong, Ming-Hsuan Yang, Irfan Essa, David A. Ross, and Lu
Jiang.
Language model beats diffusion - tokenizer is key to visual
generation.
In *ICLR*, 2024.
- Zeng et al. [2023]
Yifei Zeng, Yuanxun Lu, Xinya Ji, Yao Yao, Hao Zhu, and Xun Cao.
Avatarbooth: High-quality and customizable 3d human avatar
generation.
*arXiv preprint arXiv:2306.09864*, 2023.
- Zhang et al. [2024a]
Cheng Zhang, Yuanhao Wang, Francisco Vicente Carrasco, Chenglei Wu, Jinlong
Yang, Thabo Beeler, and Fernando De la Torre.
FabricDiffusion: High-fidelity texture transfer for 3d garments
generation from in-the-wild images.
In *ACM SIGGRAPH Asia*, 2024a.
- Zhang et al. [2023]
Lvmin Zhang, Anyi Rao, and Maneesh Agrawala.
Adding conditional control to text-to-image diffusion models.
In *ICCV*, pages 3836–3847, 2023.
- Zhang et al. [2024b]
Longwen Zhang, Ziyu Wang, Qixuan Zhang, Qiwei Qiu, Anqi Pang, Haoran Jiang, Wei
Yang, Lan Xu, and Jingyi Yu.
Clay: A controllable large-scale generative model for creating
high-quality 3d assets.
*TOG*, 43(4):1–20, 2024b.
- Zhang et al. [2024c]
Yuqing Zhang, Yuan Liu, Zhiyu Xie, Lei Yang, Zhongyuan Liu, Mengzhou Yang,
Runze Zhang, Qilong Kou, Cheng Lin, Wenping Wang, et al.
Dreammat: High-quality pbr material generation with geometry-and
light-aware diffusion models.
*TOG*, 43(4):1–18, 2024c.
- Zhao et al. [2024]
Yue Zhao, Yuanjun Xiong, and Philipp Krähenbühl.
Image and video tokenization with binary spherical quantization.
*arXiv preprint arXiv:2406.07548*, 2024.
- Zheng et al. [2024]
Yang Zheng, Qingqing Zhao, Guandao Yang, Wang Yifan, Donglai Xiang, Florian
Dubost, Dmitry Lagun, Thabo Beeler, Federico Tombari, Leonidas Guibas, et al.
Physavatar: Learning the physics of dressed 3d avatars from visual
observations.
In *ECCV*, 2024.
- Zhou et al. [2025]
Feng Zhou, Ruiyang Liu, Chen Liu, Gaofeng He, Yong-Lu Li, Xiaogang Jin, and
Huamin Wang.
Design2garmentcode: Turning design concepts to tangible garments
through program synthesis.
In *CVPR*, 2025.

## 6 Representation Details

Extended Representation. The binary concrete representations of different edge types, attachment types, and stitches are depicted in [Fig. 9](https://arxiv.org/html/2412.14453v2#S6.F9) alongside their corresponding annotations.
For edges, in addition to the vector $V_{i,j}$ representing from the start point to the endpoint, the cubic line employs the control parameters $C^{b}_{i,j}\in\mathbb{R}^{4}$ to define two control points $(x_{1},y_{1})$ and $(x_{2},y_{2})$ in the 2D coordinate.
The circle line uses additional control parameters $C^{r}_{i,j}\in\mathbb{R}^{3}$, which specify the radius $r$ and four rotations with two binary flags, including the counterclockwise acute angle ($[0,0]$), the clockwise acute angle ($[0,1]$), the counterclockwise reflex angle ($[1,0]$), and the clockwise reflex angle ($[1,1]$).
Furthermore, edge types are denoted as follows: $E^{t}_{i,j}=[0,0]$ for the straight line, $E^{t}_{i,j}=[0,1]$ for the quadratic line, $E^{t}_{i,j}=[0,0]$ for the cubic line, and $E^{t}_{i,j}=[1,1]$ for the circle line.
The attachments are visually distinguished by highlighting the associated edges in red and annotating them with the name and value of $A_{i,j}$.
Edges without attachment are not highlighted and use the default value of $A_{i,j}=[0,0,0]$.
There are six kinds of attachment types, *i.e*., lower interface ($[0,0,1]$), right collar ($[0,1,0]$), left collar ($[0,1,1]$), strapless top ($[1,0,0]$), right armhole ($[1,1,0]$), and left armhole ($[1,0,1]$).
For reversal stitch $\{M_{i,j,2}^{{}^{\prime}}\in\{0,1\}\}$, 0 means the stitch direction does not need reversal, while 1 means the stitch direction needs to be reversed.
With the detailed representation of sewing patterns, users can convert the sewing patterns into vector representations as input for neural networks.

Figure: Figure 9: Representation details. We present various kinds of edges, attachments, and stitches with detailed annotations.
Refer to caption: x7.png

Panel Order invariance.
Since all panels in the dataset are fixed, we define an order from the upper body to the lower body to construct the vector representation, guaranteeing the panel order invariance.
The panel order is represented as below:

## 7 Comparison with Parametric Method

Figure: Figure 10: Comparison with parametric method. We present the garments and draping results for our SewingLDM and parametric method GarmentCode [28]. GarmentCode needs a delicate selection of values. In contrast, our method can generate garments under intuitive conditions like natural language or sketches, which provide an easier way for garment generation.
Refer to caption: x8.png

Except for generation methods, GarmentCode [28] allows users to model complex garments by selecting different parameters and producing desired sewing patterns.
However, selecting various parameters is not intuitive and requires professional knowledge of garment design, limiting its widespread promotion.
To enable the production of the desired sewing patterns through users’ prompts, an easy way is to leverage the powerful ability of a large language model, like GPT-4 [1].
We simply ask GPT4 to generate various values between 0 to 1 to satisfy the sewing pattern designs of [28], as illustrated in [Fig. 10](https://arxiv.org/html/2412.14453v2#S7.F10).
With the designed prompt, GPT-4 can truly provide instructions for garment design.
However, most of the generated values are not concerned with garment shape in GarmentCode [28] and still require professional knowledge of garment designs and pre-defined templates.
In summary, GarmentCode [28] needs indispensable manual processing to produce the desired garments.
In contrast, our SewingLDM can generate the desired garment through more intuitive conditions, *i.e*., text prompts and sketches, providing easier tools for garment designs and boosting daily garment production.

Figure: Figure 11: Comparison of sewing pattern generation for concurrent works. Since all the methods were trained on the same dataset, we randomly selected samples from the original dataset to evaluate and compare their performance.
Refer to caption: extracted/6600879/figs/comparison.jpg

## 8 Concurrent Work Comparison

For concurrent sewing pattern generation methods, both Design2GarmentCode [76] and ChatGarment [7] use autoregressive models to convert inputs into rule-based parameters specially designed in GarmentCode, limiting their generalization beyond pre-defined garment categories.
AIpparel [46], enabling the generation of any extended sewing patterns without requiring category-specific parameterization, while it only considers the average body shape draping.
We also compared with these open-sourced methods, ChatGarment [7] and AIpparel [46].
Since all methods are trained on the GarmentCodeData [29], we randomly chose some examples in the GarmentCodeData for fair comparison, shown in [Fig. 11](https://arxiv.org/html/2412.14453v2#S7.F11).
ChatGarment directly generates the design files and uses GarmentCode to generate appropriate sewing patterns and drape to other body shapes.
AIpparel generates sewing patterns and warps to the average body shapes and drapes to other body shapes by complete pattern scaling manually.
Compared with concurrent works, our SewingLDM shows comparable results with ChatGarment and shows much superior results with AIpparel.
Moreover, due to the body shape conditions, our method can fit various body shapes, which provides a more user-friendly approach to getting the desired made-to-measure garments.

## 9 Dataset Annotation Details

Directly employing multimodal models for generating textual descriptions of images may lead to inaccuracies, as these large-scale multimodal models are incapable of fully capturing and characterizing garment-specific features, often resulting in hallucination phenomena.
To obtain precise descriptions of garment characteristics, we have established a corresponding descriptive framework based on the parameters of a parametric model, generating distinct descriptions for various garment features, which are then concatenated together to form the text annotations of garments.
The corresponding descriptions are presented as follows:

In addition to characterizing apparel attributes based on numerical data, we also provide corresponding descriptions according to the categories of different garments, for instance, shirt, fitted shirt, straight waistband, fitted waistband, pencil skirt, circle skirt, mermaid skirt, tail dress, etc.
Here is an example of garment description: Upper garment: asymmetrical fitted shirt, right: curve armhole, ruffled shoulder, wide short ruffled sleeve, short shrink cuffs, front wide shallow V neckline and back wide shallow V neckline; left: strapless; Lower garment: ankle length circle skirt with front right deep split.
After getting the detailed description of the garments, we use GPT-4 to refine them, allowing for more diverse descriptions and more general prompt queries.
The prompt used to refine them is shown in [Tab. 4](https://arxiv.org/html/2412.14453v2#S9.T4).
For each garment, we utilize PiDiNet [57], a pre-trained edge detection network, to extract garment sketches using its open-source code and checkpoint, thereby enriching the design details of the garment.

Table: Table 4: Prompt to refine the dataset annotations.

## 10 Extended Vector v.s Origin Vector

Figure: Figure 12: Visulization of different representation. We compare the extended representation with the original representation.
Refer to caption: x9.png

We also conduct the visualization between the extended representation and original representation to reveal significant advantages in design versatility of the extended representation shown in [Fig. 12](https://arxiv.org/html/2412.14453v2#S10.F12).
With extended representation, our model can generate more diverse garments and more fashion designs, such as the curve neckline and the tail dress.
In contrast, the original representation restricts the model to producing only simple garments, failing to capture complex design features and resulting in a notable loss of detail.
To address these limitations, the integration of the GarmentCodeData is essential, as it injects detailed knowledge of complex sewing patterns into the generation model.
This advancement overcomes the constraints of previous methods, which were unable to represent the extended vector effectively due to more than 30k tokens consumption under current GPU capabilities.

## 11 One-step Training v.s Two-step Training

Figure: Figure 13: Ablation on the training strategy. One-step training shows an unbalance between the multi-modal conditions, failing under only the sketch. In contrast, two-step training helps to faithfully generate the ideal garments under only sketch conditions.
Refer to caption: x10.png

We additionally conduct an ablation study on the training strategy.
One-step training is unable to balance the multi-modal conditions, so that fails to generate the corresponding garment through only the sketch condition.
As shown in [Fig. 13](https://arxiv.org/html/2412.14453v2#S11.F13), the generated garment loses its midi-length pencil skirt, failing to generate the desired garment.
In contrast, the model under two-step training can faithfully generate the corresponding garment with the sketch only, which contains both the sleeveless fitted shirt and the corresponding midi-length pencil skirt.
Therefore, two-step training can more effectively inject the sketch conditions into the diffusion model and provide additional control of garment designs, enabling wider usage of our SewingLDM.
Moreover, combined with full conditions of text and sketch, SewingLDM can provide more precise control over desired garment generations, meeting users’ requirements.

## 12 User Study Details

Figure: Figure 14: User study examples. We present 3 user study examples with random shuffled results.
Refer to caption: x11.png

To ensure a fair and objective evaluation of our method compared to other methods, we randomly shuffle the results generated by different methods.
Each result is paired with a corresponding textual description, and volunteers are asked to rate the results with a score of $1-5$ based on the consistency between the results and the texts, as well as the fitness between the clothes and the human bodies.
Additionally, we provide 3 supplementary examples as shown in [Fig. 14](https://arxiv.org/html/2412.14453v2#S12.F14).

## 13 Use Case

Figure: Figure 15: Use case. We present an example of an extremely complex sewing pattern generation. With the generated sewing pattern, we can easily paint the UV to produce visually appealing results.
Refer to caption: extracted/6600879/figs/user_case.png

Our SewingLDM is capable of generating intricately detailed clothing, meeting the current artistic demands for garment design across a wide range of styles, significantly advancing fashion garment design, and supporting everyday users in obtaining apparel tailored precisely to their needs.
To demonstrate our superiority in garment generation, we present an extremely complex example of a sewing pattern in [Fig. 15](https://arxiv.org/html/2412.14453v2#S13.F15).
With the detailed textual description and garment sketch, our method faithfully generates the complex sewing pattern, *e.g*., skirt band cuff, hood, and godets, which significantly helps the artist in creating fantastic texture in UV space, *e.g*., laces, leather pants, and hat brim.

## 14 Qualitative Results

For in-domain garment sketches, our method achieves a more granular representation that closely adheres to the sketch contours.
We have also provided a comparative analysis between our approach and other baseline methods within the same domain, as illustrated in [Fig. 16](https://arxiv.org/html/2412.14453v2#S14.F16). Furthermore, to demonstrate the efficacy of our method, we have conducted additional validation using out-of-domain data that are collected from the Multimodal Garment Designer or hand-drawn, as depicted in [Fig. 17](https://arxiv.org/html/2412.14453v2#S14.F17) and [Fig. 18](https://arxiv.org/html/2412.14453v2#S14.F18).

Figure: Figure 16: Additional results for in-domain sketches. We present the in-domain comparison with baseline methods.
Refer to caption: x12.png

Figure: Figure 17: Additional results for in-the-wild sketches. We also provide more results for in-the-wild sketches.
Refer to caption: x13.png

Figure: Figure 18: Additional results for hand-drawn sketches.
Refer to caption: x14.png

We additionally provide garments tailored to a wide range of body shapes, spanning variations such as short to tall and slim to broad. As illustrated in [Fig. 19](https://arxiv.org/html/2412.14453v2#S14.F19), our approach enables the creation of garments specifically adapted to different body types.
Furthermore, the simulated garments are enriched with physically based rendering (PBR) textures, either generated by DressCode or designed using the Substance 3D Painter software [2], culminating in visually compelling garment representations as shown in [Fig. 20](https://arxiv.org/html/2412.14453v2#S14.F20).

Figure: Figure 19: Additional results for various body shapes. We present identical garment designs tailored for two distinct body types, encompassing a spectrum of heights and body compositions, to demonstrate the effectiveness of our SewingLDM across diverse bodies.
Refer to caption: x15.png

Figure: Figure 20: Additional qualitative results. By integrating Physically Based Rendering (PBR) textures, our generated outputs achieve visually compelling rendering effects, particularly for a wide range of intricate garment designs.
Refer to caption: x16.png