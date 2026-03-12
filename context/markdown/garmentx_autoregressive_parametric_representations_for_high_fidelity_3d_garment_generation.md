<!-- markdownlint-disable -->
## Contents
- 1 Introduction
- 2 Related work
  - Open Surface and Garment Pattern Generation
  - Deformation-based 3D Garment Generation.
  - Autoregressive Models for 3D Generation.
- 3 Method
  - 3.1 GarmentX Parameter Representation
  - 3.2 GarmentX Dataset Construction
  - 3.3 3D Garment Generation
- 4 Experiments
  - 4.1 Datasets, Metrics, and Implementation Details
  - 4.2 Comparisons
  - 4.3 Ablation Study
  - 4.4 Garment Editing
- 5 Conclusion and Future work
- References
- 6 More Implementation Details
- 7 More Qualitative Results
- 8 Future Works
- 9 Failure Cases

## Abstract

Abstract This work presents GarmentX, a novel framework for generating diverse, high-fidelity, and wearable 3D garments from a single input image. Traditional garment reconstruction methods directly predict 2D pattern edges and their connectivity, an overly unconstrained approach that often leads to severe self-intersections and physically implausible garment structures. In contrast, GarmentX introduces a structured and editable parametric representation compatible with GarmentCode, ensuring that the decoded sewing patterns always form valid, simulation-ready 3D garments while allowing for intuitive modifications of garment shape and style.
To achieve this, we employ a masked autoregressive model that sequentially predicts garment parameters, leveraging autoregressive modeling for structured generation while mitigating inconsistencies in direct pattern prediction. Additionally, we introduce GarmentX dataset, a large-scale dataset of 378,682 garment parameter-image pairs, constructed through an automatic data generation pipeline that synthesizes diverse and high-quality garment images conditioned on parametric garment representations. Through integrating our method with GarmentX dataset, we achieve state-of-the-art performance in geometric fidelity and input image alignment, significantly outperforming prior approaches. We will release GarmentX dataset upon publication.

## 1 Introduction

3D Garments play a crucial role in digital humans [24, 25, 56, 10, 9, 57, 62, 70, 61, 19, 32], and virtual try-on applications [59, 16, 14, 15], serving as essential assets in film VFX, game development, and AR/VR experiences. Recent 3D generation approaches [35, 39, 69, 68, 26, 65] have provided valuable guidance for transforming how digital clothing items can be created with high-fidelity and physically plausible results. Traditional 3D garment creation pipelines primarily rely on two approaches: (1) manual modeling using professional software like CLO3D [3] and (2) high-fidelity 3D scanning system 3dMD [1] and Artec3D [2] that capture garment geometry through multi-camera photogrammetry. However, both methods require either specialized expertise or costly hardware, creating prohibitive barriers for large-scale 3D garment production.

Recently, deep learning methods have been explored for text-to-garment and image-to-garment generation. While text-based methods [22, 31, 37, 50] provide high-level control, they struggle to capture fine-grained garment details such as sleeve width, skirt flare, or neckline shape. Image-based methods offer richer details but still face severe limitations in structural accuracy and editability.
The image-to-garment generation direction can be categorized into two types: pattern-based methods [8, 36, 46, 43, 5] and deformation-based approaches [48, 13, 40].
Pattern-based approaches predict sewing pattern edges, stitch connectivity, and 2D panel relationships. However, this formulation is overly unconstrained, often leading to self-intersections, misaligned panels, and physically invalid patterns.
Deformation-based methods rely on predefined 3D garment templates, which are warped to fit the target shape. However, these methods are highly dependent on the chosen template, making them prone to misalignment when dealing with unseen garment styles or out-of-distribution shapes.
Meanwhile, general image-to-3D approaches [60, 52, 58, 54] can produce matching results, but these results often contain closed or double-layered structures, making them unwearable.

To address these challenges, we introduce GarmentX, a parametric driven framework for structured, editable, and physically valid 3D garment generation.
Unlike previous approaches that directly predict sewing pattern edges or deform template meshes, GarmentX represents garments through a structured parametric representation that defines high-level, semantically meaningful attributes.
By predicting garment parameters instead of raw pattern edges, GarmentX ensures that the generated garments maintain correct stitching relationships, geometric integrity, and simulation robustness. This structured formulation not only enhances the quality of the generated garments but also makes them highly controllable and editable.

Given the unordered nature of garment parameters and the inherent advantages of autoregressive models in visual generation—particularly their superior scalability and generation speed— we leverage a masked autoregressor for garment parameter generation.
By employing an autoregressive model, we decompose the complex joint distribution modeling into a product of multiple simpler marginal distributions, making generation more tractable. Our autoregressor incorporates a diffusion process to effectively handle the continuous values of garment parameters, enabling more precise and flexible customization.

To support training and evaluation, we construct GarmentX dataset, a data collection of 378,682 garment parameter-image pairs, built using an automatic data construction pipeline. Unlike prior datasets, which rely on low-level pattern descriptors, we transform GarmentCodeData’s [30] raw attributes into GarmentX structured representation, ensuring that garments remain semantically meaningful and editable.
To bridge the domain gap between synthetic rendering and real-world garments, we further employ a canny-conditioned ControlNet model [66] to synthesize high-quality, photorealistic garment images, improving the robustness of the learned garment representation.
Extensive experiments show that GarmentX outperforms state-of-the-art methods in geometry fidelity, input-image alignment, and wearability. We summarize our contributions:

- •
We introduce GarmentX, a novel masked autoregressive framework that models garments through a structured and editable parametric representation, ensuring controllable and physically valid garment generation.
- •
We build and release GarmentX dataset, a collection of 378,682 garment parameter-image pairs generated by an automatic data construction pipeline.
The dataset provides diverse, high-fidelity garment images aligned with parametric descriptions.
- •
We achieve the state-of-the-art results in single-view garment generation on several benchmarks, with particular improvements in geometric fidelity, generalization capability, and input-image alignment.

## 2 Related work

#### Open Surface and Garment Pattern Generation

Open surface generation has advanced substantially through unsigned distance fields (UDFs) [63, 38]. For clothing, sewing patterns provide a superior approach by precisely representing garment structure without the difficult gradient predictions of UDFs. Pattern-based techniques predict panel and stitch data before 3D simulation. Notable advances include NeuralTailor’s [28] LSTM-based pattern reconstruction, transformer architectures from SewFormer [36] and PanelFormer [8] for image-to-pattern conversion, and DressCode’s [22] GPT-based text-conditioned pattern generation. These methods predict the panel information through its rotational matrix, translation matrix, vertex coordinates, edge connection relationships, edge types, and curvature, with stitches defined as a set of one-to-one connections between the edge of one panel and the edge of another panel. While effective for simple garments, they struggle with complex designs due to the combinatorial explosion in pattern token sequences. Generated patterns often mismatch input, with issues such as self-intersections or incorrect stitching information, leading to simulation failures. Recent work mitigates these through large multimodal models [5] and a novel tokenization scheme [43]. Our key insight diverges intrinsically: instead of directly predicting panel and stitch information, we focus on generating higher-level garment parameters, which are more intuitive elements and are easier to obtain from images.

#### Deformation-based 3D Garment Generation.

These methods typically take a pre-provided template garment as input and deform it to match the target garment using implicit fields and score distillation sampling (SDS). GarmentDreamer [31] and ClotheDreamer [37] employ text-guided deformation of 3D Gaussian Splatting (3DGS) representations. Garment3DGen [48] introduces geometric supervision in 3D space by generating a coarse guidance mesh from image input and using it as a soft constraint during the deformation process. WordRobe [50] designs a garment latent space and aligns it with the CLIP embedding space to enable text-guided garment generation and editing. DiffAvatar [34] further optimize the garment to match the observation with proposed efficient differentiable simulation. While benefiting from recent advances in differentiable rendering and simulation, they still require lengthy optimization or deformation processes and exhibit strong dependency on the quality of the pre-provided template. Such limitations fundamentally restrict their adoption in real-time applications and large-scale production.

#### Autoregressive Models for 3D Generation.

Auto-regressive transformers [55, 20] have radically transformed visual generation [17, 6, 7, 53, 51, 33, 11, 18, 41] through their sophisticated sequential approach of synthesizing images using discrete tokens derived from image tokenizers. This paradigm has achieved remarkable success by decomposing the complex task of image generation into a series of manageable token prediction steps, enabling more coherent and controllable outputs.
Building upon this foundation, recent works: SAR3D [12] and MAR-3D [11] effectively discretize 3D geometry into sequential tokens that can be predicted in an auto-regressive manner, similar to language modeling. More specifically, SAR3D developed a multi-scale quantization pipeline and uses VAR as the generation backbone. While MAR-3D combines the power of auto-regressive model and diffusion model to enable high-resolution mesh generation while maintaining computational efficiency. Our method builds upon these advances, first leveraging the strengths of auto-regressive models for garment parameters generation while using diffusion loss [33] that accounts for the continuous value space inherent in garment parameters, results in more realistic and physically plausible garment generations.

## 3 Method

We propose GarmentX, a structured and parametric-driven framework for generating diverse, editable, and simulation-ready 3D garments from a single input image.
Central to our approach is a structured garment parameter representation that ensures physically valid and simulation-ready garment synthesis (Section [3.1](https://arxiv.org/html/2504.20409v1#S3.SS1)). To enable effective learning, we construct GarmentX dataset, a large-scale dataset containing 378,682 garment parameter-image pairs, created through a novel automatic data construction pipeline based on the GarmentCodeData dataset (Section [3.2](https://arxiv.org/html/2504.20409v1#S3.SS2)).
To generate structured garment parameters from an input image, we employ a masked autoregressive model to efficiently decode conditional image tokens into our representation.
Once generated, these garment parameters are transformed into 2D sewing patterns and simulated to 3D garments through GarmentCode (Section [3.3](https://arxiv.org/html/2504.20409v1#S3.SS3)). We provide an overview of the proposed framework in Figure [3](https://arxiv.org/html/2504.20409v1#S3.F3).

Figure: Figure 2: Automatic Data Construction Pipeline. We construct garment parameters-image pairs using ControlNet and Blender.
Refer to caption: extracted/6397419/imgs/data_pipline.png

Figure: Figure 3: Overview of GarmentX. Taking a single image as input, GarmentX trains a masked autoregressive generation model directly upon our GarmentX representation, extracts condition image tokens via DINOv2, processes them through MAE encoder-decoder architecture and diffusion MLP. The generated GarmentX representation is projected back to original scale and then reconstruct sewing patterns through GarmentCode, and finally simulate 3D garments wearable on arbitrary human bodies.
Refer to caption: extracted/6397419/imgs/garment_pipline.png

### 3.1 GarmentX Parameter Representation

A key innovation of GarmentX is the introduction of a structured and editable parametric representation for garments. Instead of directly predicting 2D pattern edges and stitch relationships, which often results in geometrically invalid configurations, our approach represents garments using a structured parameter vector that can be decoded into a valid sewing pattern via GarmentCode [29].

Each garment is represented by a parameter vector $p=\left\{{{p_{0}},{p_{1}},\ldots,{p_{N}}}\right\}$, where each ${p_{i}}$ corresponds to an editable garment attribute, such as neckline type, sleeve shape, pants length, and skirt flare.
This structured parameterization enables direct control over garment properties, facilitating both accurate synthesis and interactive editing.

To preserve relative positional relationships while ensuring numerical stability during diffusion training, the parameter values are normalized to a fixed range $\left[{-1,1}\right]$. For continuous parameters, normalization is performed using:

$$ ${x_{i}}=2\cdot\frac{{{p_{i}}-{p_{i\min}}}}{{{p_{i\max}}-{p_{i\min}}}}-1,$ (1) $$

where $p_{i\min}$, $p_{i\max}$ and ${x_{i}}$ denote the minimum value, maximum value, and normalized value of the i-th parameter.
For discrete attributes with $K$ ordinal choices, such as neckline type or sleeve style, we map each category to a continuous space uniformly partitioning $\left[{-1,1}\right]$ into $K$ intervals of width $\frac{2}{K}$.
Each discrete option $k\in\left\{{0,1,\ldots K}\right\}$ is encoded as the midpoint of its corresponding interval:

$$ ${x_{i}}=\frac{{2\cdot\left({k+0.5}\right)}}{K}-1$ (2) $$

For instance, when $K=3$, the discrete choices $\left\{{A,B,C}\right\}$ map to $\left\{{-0.666,0,0.666}\right\}$ respectively, creating a continuous space compatible with neural network operations.
The normalization operation can be expressed as $x=\mathbb{N}\left(p\right)$. During inference, the generated garment parameters are projected back to the original scale, represented as $p={\mathbb{N}^{-1}}(x)$. The 3D garments are then generated using parameter $p$.
This structured representation ensures consistent scaling across different garment attributes, improving training stability and generalization.

### 3.2 GarmentX Dataset Construction

A key requirement for training GarmentX is a large-scale high-quality dataset containing paired garment images and our structured garment parameters.
However, existing datasets often lack fine-grained parametric annotations or are limited in garment diversity, restricting the generalization ability of learning-based methods.
To accommodate this need, we build GarmentX dataset, a collection of 378,682 garment parameter-image pairs, using an automatic data construction pipeline that synthesizes realistc garment images aligned with GarmentX parameter representation.

To ensure compatability with our framework, we transform the raw garment parameters from GarmentCodeData into our GarmentX representation.
GarmentCodeData provides low-level garment attributes, such as panel dimensions, edge connectivity, and stitch relationships, these parameters are often difficult to manipulate and prone to self-intersections when directly used for prediction.
By converting GarmentCodeData parameters into GarmentX representations, we ensure that every sampled parameter vector is guaranteed to generate physically valid garments when decoded back into 2D sewing patterns and 3D simulations.

As shown in Figure [2](https://arxiv.org/html/2504.20409v1#S3.F2), given a set of 3D garments assets $G$, we render the multi-view images by randomly sampling camera poses from the combinations of three azimuths [0, 30, -30] and three elevations [0, 30, -30]. Once the multi-view images are generated, we enhance their realism using a canny-conditioned ControlNet [66], which transforms the rendered images into photorealistic counterparts. The base prompt “A modern garment” ensures the generation of realistic, contemporary clothing styles, while supplementary descriptors such as “best-quality,” “realistic,” and “photoreal” further enhance image fidelity.
Through this automated pipeline, we construct GarmentX dataset, a large-scale collection containing 378,682 well-aligned garment parameter-image pairs.

### 3.3 3D Garment Generation

Our framework generates editable and physically valid 3D garments by predicting structured garment parameters through a masked autoregressive model. These parameters are then converted into sewing patterns and simulated into wearable 3D garments, ensuring realism, diversity, and controllability.

Condition Scheme.
We use DINOv2 [44] to extract image features, which serve as initial tokens for the masked autoencoder (MAE [21]) encoder. To enhance conditional generation quality, we apply classifier-free guidance (CFG), randomly nullifying conditional input features with a 0.1 probability during training [23].

MAE Encoder and Decoder.
The integration of image tokens and normalized GarmentX representations is achieved through concatenation, followed by random masking at a variable ratio of 0.7 to 1.0. In accordance with the MAE architectural principles, bidirectional attention mechanisms are implemented. The computational pipeline starts with the unmasked tokens through the MAE encoder, which comprises a series of self-attention layers. Subsequently, the encoded tokens are concatenated with the mask tokens with learnable positional embeddings.

Diffusion Process.
For each token utilized in the diffusion process, the MAE decoder generates a corresponding condition vector $\mathbf{z}\in\mathbb{R}^{D}$. A MLP-based denoising network subsequently reconstructs ground truth tokens from Gaussian noise through the optimization of:

$$ $\mathcal{L}(z,x)=\mathbb{E}_{z,t}\left[|\epsilon-\epsilon_{\theta}(x^{t}|t,z)| ^{2}\right]$ (3) $$

where $z$ represents the condition vector derived from the MAE decoder and $x_{t}$ denotes the ground truth parameters. During the inference phase, the reverse diffusion process is employed to predict each set of tokens in parallel.

Inference Schedules.
During inference, we first generate a random token generation order. Then we extract image tokens from input image, which are fed into the MAE encoder-decoder architecture. From the decoder outputs, we select condition vector $z$ according to the predetermined generation order. Multiple tokens undergo parallel DDIM [49] denoising processes simultaneously. The number of tokens $N_{s}$ processed in each auto-regressive step follows a cosine schedule as in [6], progressively increasing over $S$ total steps:

$$ $\displaystyle N_{s}=\left\lfloor N(cos(\frac{s}{S})-cos(\frac{s-1}{S}))\right\rfloor$ (4) $$

This scheduling strategy is motivated by the observation that initial tokens are more challenging to predict, while later tokens become progressively easier to determine, similar to a completion task. Consequently, we generate fewer tokens in initial steps and gradually increase the number in later steps, rather than maintaining a constant generation rate across all steps.

CFG Schedule. We employ CFG in our diffusion model:

$$ $\displaystyle\varepsilon=\varepsilon_{\theta}(x_{t}|t,z_{u})+\omega_{s}\cdot( \varepsilon_{\theta}(x_{t}|t,z_{c})-\varepsilon_{\theta}(x_{t}|t,z_{u}))$ (5) $$

where $z_{c}$ and $z_{u}$ are conditional and unconditional output from the MAE decoder, which serve as the condition for the diffusion model. We employ a linear strategy [33] for the CFG coefficient $\omega_{s}$, starting with lower values during the initial uncertain steps and progressively increasing it to $\lambda_{cfg}$. Specifically, in Eq. [5](https://arxiv.org/html/2504.20409v1#S3.E5), we set $\omega_{s}=s\cdot\lambda_{cfg}/S$.

3D Garment Simulation.
Given the generated garment parameters, we leverage the established GarmentCode to first produce corresponding sewing patterns, followed by physically simulating 3D garments wearable on arbitrary human bodies. The strictly constrained parameter boundaries eliminate erroneous pattern generation observed in previous pattern-based approaches, thereby preventing simulation failures inherent to prior methods.

## 4 Experiments

**Table 1: Quantitative comparisons on CLOTH3D dataset. GarmentX achieves the best performance.**
| Method | CD $\downarrow$ | P2S $\downarrow$ | Failure Rate $\downarrow$ | Runtime $\downarrow$ | Wearable |
| --- | --- | --- | --- | --- | --- |
| SewFormer | $9.70$ | $10.14$ | 76.3% | $2$min | ✓ |
| LGM | $9.51$ | $7.68$ | - | $82$s | ✗ |
| Garment3DGen | $6.34$ | $5.91$ | - | $15$min | ✓ |
| InstantMesh | $5.99$ | $5.70$ | - | $20$s | ✗ |
| Ours | $4.92$ | $4.96$ | 0 0 | $15$s | ✓ |

### 4.1 Datasets, Metrics, and Implementation Details

Datasets. We use GarmentX dataset for training and the validation set of CLOTH3D [4] for qualitative and quantitative comparison with baselines.
Due to the limited garment styles in CLOTH3D, especially for fashion dresses, we supplemented our qualitative evaluation with the recently proposed Complex Virtual Dressing Dataset (CVDD) [27], which contains diverse and intricate fashion garments.

Metrics. We employ Chamfer Distance (CD), Point-to-Surface distance (P2S), simulation failure rate, and runtime as quantitative metrics. We also record whether each method produces wearable results to evaluate its suitability for downstream tasks.

Figure: Figure 4: Qualitative comparisons on CLOTH3D dataset. GarmentX produce wearable, open-structure, and complete 3D garments.
Refer to caption: extracted/6397419/imgs/comp_CLOTH3D.png

Implementation Details. Input images are resized into $224\times 224$ and processed by the DINOv2 ViT-B/14 model to extract condition tokens. Training completes in 50 hours on two GPUs with a batch size of 48. More details are provided in supplementary materials.

### 4.2 Comparisons

We evaluate our approach against three types of methods: 1) pattern-based methods, including SewFormer [36] and DressCode [22], which directly predict the panel and stitch information of sewing pattern; 2) deformation-based methods, such as Garment3DGen [48]; and 3) general 3D generation methods, including InstantMesh [60], LGM [52] and Trellis [58]. Since DressCode is a text-guided method, to ensure fairness, we compare it only from the perspective of generalization performance.

Quantitative Comparisons. Table [1](https://arxiv.org/html/2504.20409v1#S4.T1) presents quantitative comparisons between our method and baseline approaches. Our method performs best on both the CD and P2S metrics, surpassing the strongest competitor InstantMesh by margins of 1.07 in CD and 0.74 in P2S metrics. This demonstrates its superior capability in accurately generating 3D garments aligned with input images. Previous direct sewing pattern prediction approaches like SewFormer exhibit fundamental limitations—their 76.3% simulation failure rate stems from frequent self-intersection panels and erroneous stitch predictions. Our method fundamentally resolves these issues, achieving zero failure cases. Additionally, our method maintains efficient inference time of 15 seconds per generation, a 60$\times$ speed improvement over deformation-based methods such as Garment3DGen, which takes 15 minutes per sample.

Wearable. While InstantMesh and LGM are competitive in quantitative metrics, they tend to produce closed garments that cannot be worn directly, as evidenced by the side and top views in Figure [4](https://arxiv.org/html/2504.20409v1#S4.F4). Furthermore, as evidenced by cross-sectional analysis through mesh cutting operations in MeshLab, shown in Figure [5](https://arxiv.org/html/2504.20409v1#S4.F5), Trellis generates double-layer meshes with some sticky areas at the ends of skirts, and it completely closes when dealing with the T-shirt images. In contrast, our method produces single-layer, open-structure garments, which can be directly draped onto various human bodies and seamlessly animated through simulators.

Qualitative Comparisons. Figure [4](https://arxiv.org/html/2504.20409v1#S4.F4), [5](https://arxiv.org/html/2504.20409v1#S4.F5), and [6](https://arxiv.org/html/2504.20409v1#S4.F6) provide qualitative comparisons between our method and baseline approaches. SewFormer exhibits limited generalization capability beyond the training data distribution, thereby constraining reconstructable garment diversity, and frequently resulting in geometrically distorted garments due to flawed stitches. Garment3DGen’s performance is heavily dependent on pre-provided 3D garment templates, leading to highly mismatched or even incomplete results when the template and input image differ substantially. In contrast, our method demonstrates precision in reconstructing nuanced features, as shown in Figure [6](https://arxiv.org/html/2504.20409v1#S4.F6), such as irregular ribbons (a, g), collar (b, d, e), shoulder belt (c), and waistline (f), while the baseline methods fail to reproduce these details. More results are included in the supplementary materials.

Figure: Figure 5: Qualitative comparisons with Trellis. GarmentX generates single-layer, open-structure 3D garments.
Refer to caption: extracted/6397419/imgs/comp_Trellis.png

Figure: Figure 6: Qualitative comparisons on CVDD dataset. GarmentX produce diverse, complex, and detailed 3D garments that match the input images well.
Refer to caption: extracted/6397419/imgs/comp_CVDD.png

Generalization. We also evaluate the generalization capabilities of our method in comparison to DressCode, with comparative results visualized in Figure [7](https://arxiv.org/html/2504.20409v1#S4.F7). Our approach demonstrates superior performance in generating diverse fashion garments, such as asymmetric one-shoulder dresses, top harnesses, and strapless dresses, which DressCode fails to produce. Additionally, as evidenced in Figure [8](https://arxiv.org/html/2504.20409v1#S4.F8), despite never being trained on sketch images, our model consistently generates 3D garments that maintain precise geometric alignment with the input sketch contours.

Figure: Figure 7: Qualitative comparisons with DressCode. GarmentX exhibits better generalization, enabling the creation of a variety of uncommon garments.
Refer to caption: extracted/6397419/imgs/comp_DressCode.png

Figure: Figure 8: Sketch-conditional inputs. Note that GarmentX has not been trained on sketch images.
Refer to caption: extracted/6397419/imgs/comp_Sketch.png

### 4.3 Ablation Study

Choice of Data Construction Methods. There are two possible approaches for obtaining a garment image from a 3D garment mesh: render first and then generate the image, or generate textures first and then render it. For the first approach, we compared two canny-conditional Stable Diffusion models: ControlNet and T2I-Adapter [42]. For the second approach, we generated textures with Paint3D [64] or DreamMat [67] and then rendered them. We chose the first method because texture generation takes 5 minutes with Paint3D and 20 minutes with DreamMat, making it difficult to conduct large-scale data construction, whereas ControlNet or T2I-Adapter can generate images in 5 seconds. We ultimately select ControlNet due to T2I-Adapter producing lower quality images that often failed to resemble garments, as shown in Figure [9](https://arxiv.org/html/2504.20409v1#S4.F9).

Autoregressive model vs DiT [45]. To verify the accuracy of our autoregressor in generating garment parameters, we conducted experiments by replacing our model with DiT. As quantitatively demonstrated in Table [2](https://arxiv.org/html/2504.20409v1#S4.T2), our method achieves a 1.25 lower CD metric and a 1.24 lower P2S metric compared to DiT-XL, while requiring 43% fewer parameters. This advantage stems from our autoregressive formulation that decomposes complex joint distribution modeling into a product of simpler marginal distributions, enabling faster parallel generation and higher quality outputs.

Ablation on CFG Scale. To balance the generation quality and condition strength, we systematically evaluate CFG scale ${\lambda_{cfg}}\in\left[{2.0,5.0}\right]$. As Table [3](https://arxiv.org/html/2504.20409v1#S4.T3) demonstrates, $\lambda_{cfg}=3.0$ achieves the optimal trade-off, which is consequently adopted in our model.

**Table 2: Quantitative ablation of model. Our autoregressor outperforms DiT on CLOTH3D dataset and also uses a more optimal number of parameters.**
| Method | CD $\downarrow$ | P2S $\downarrow$ | num parameters |
| --- | --- | --- | --- |
| DiT-S | $6.47$ | $6.40$ | $30$M |
| DiT-B | $6.47$ | $6.35$ | $119$M |
| DiT-XL | $6.17$ | $6.20$ | $607$M |
| Ours | $4.92$ | $4.96$ | $352$M |

Figure: Figure 9: Qualitative ablation of different data construction methods. ControlNet was selected for its optimal balance between speed and quality.
Refer to caption: extracted/6397419/imgs/comp_data.png

**Table 3: Quantitative compartion of CFG scaling factor on CLOTH3D dataset. We finally adopt 3.0 in our model.**
| CFG scale | CD $\downarrow$ | P2S $\downarrow$ |
| --- | --- | --- |
| 2.0 | $5.14$ | $5.21$ |
| 3.0 | $4.92$ | $4.96$ |
| 4.0 | $5.37$ | $5.38$ |
| 5.0 | $5.03$ | $5.07$ |

### 4.4 Garment Editing

The parametric representation framework inherently facilitates geometry-aware garment customization. As shown in the lower panel of Figure [1](https://arxiv.org/html/2504.20409v1#S0.F1), our generated garment parameters enable intuitive modification of style features (e.g., removing sleeves), components design (e.g., adding a hood), and even garment category (e.g., transforming dress into a jumpsuit) through minimal parameter adjustments. This parametric representation framework significantly enhances interoperability with downstream applications such as fashion design platforms.

## 5 Conclusion and Future work

We introduce GarmentX, a structured and parametric-driven framework for editable and physically valid 3D garment generation. By leveraging a masked autoregressive model, GarmentX efficiently predicts structured garment parameters, ensuring high-fidelity garment synthesis. Additionally, we construct GarmentX dataset, a large-scale data collection of garment-parameter-image pairs, enabling robust training and improved generalization.

GarmentX shows significant advances but has limitations. It is currently trained on human-free garment images and requires such images as input. For photos with wearers, we adopt SAM2 [47] to segment clothing before generation (results are included in supplementary materials). Future work will expand the proposed automatic data construction pipeline to clothed humans, enabling direct processing without segmentation.

## References

- [1]
3dMD.
[https://3dmd.com/](https://3dmd.com/).
- [2]
Artec3D.
[https://www.artec3d.com/portable-3d-scanners/](https://www.artec3d.com/portable-3d-scanners/).
- [3]
Clo3D.
[https://clo3d.com/en/](https://clo3d.com/en/).
- Bertiche et al. [2020]
Hugo Bertiche, Meysam Madadi, and Sergio Escalera.
CLOTH3D: Clothed 3D Humans.
In *European Conference on Computer Vision*, pages 344–359. Springer, 2020.
- Bian et al. [2024]
Siyuan Bian, Chenghao Xu, Yuliang Xiu, Artur Grigorev, Zhen Liu, Cewu Lu, Michael J Black, and Yao Feng.
ChatGarment: Garment Estimation, Generation and Editing via Large Language Models.
*arXiv preprint arXiv:2412.17811*, 2024.
- Chang et al. [2022]
Huiwen Chang, Han Zhang, Lu Jiang, Ce Liu, and William T. Freeman.
MaskGIT: Masked Generative Image Transformer.
In *The IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*, 2022.
- Chang et al. [2023]
Huiwen Chang, Han Zhang, Jarred Barber, AJ Maschinot, Jose Lezama, Lu Jiang, Ming-Hsuan Yang, Kevin Murphy, William T. Freeman, Michael Rubinstein, Yuanzhen Li, and Dilip Krishnan.
Muse: Text-To-Image Generation via Masked Generative Transformers, 2023.
- Chen et al. [2024a]
Cheng-Hsiu Chen, Jheng-Wei Su, Min-Chun Hu, Chih-Yuan Yao, and Hung-Kuo Chu.
PanelFormer: Sewing Pattern Reconstruction from 2D Garment Images.
In *Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision*, pages 454–463, 2024a.
- Chen et al. [2024b]
Jinnan Chen, Chen Li, and Gim Hee Lee.
Dihur: Diffusion-guided generalizable human reconstruction.
*arXiv preprint arXiv:2411.11903*, 2024b.
- Chen et al. [2024c]
Jinnan Chen, Chen Li, Jianfeng Zhang, Lingting Zhu, Buzhen Huang, Hanlin Chen, and Gim Hee Lee.
Generalizable human gaussians from single-view image.
*arXiv preprint arXiv:2406.06050*, 2024c.
- Chen et al. [2025a]
Jinnan Chen, Lingting Zhu, Zeyu Hu, Shengju Qian, Yugang Chen, Xin Wang, and Gim Hee Lee.
Mar-3d: Progressive masked auto-regressor for high-resolution 3d generation.
*arXiv preprint arXiv:2503.20519*, 2025a.
- Chen et al. [2025b]
Yongwei Chen, Yushi Lan, Shangchen Zhou, Tengfei Wang, and Xingang Pan.
SAR3D: Autoregressive 3D Object Generation and Understanding via Multi-scale 3D VQVAE.
In *CVPR*, 2025b.
- De Luigi et al. [2023]
Luca De Luigi, Ren Li, Benoit Guillard, Mathieu Salzmann, and Pascal Fua.
DrapeNet: Garment Generation and Self-Supervised Draping.
In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition*, pages 1451–1460, 2023.
- Dong et al. [2019a]
Haoye Dong, Xiaodan Liang, Xiaohui Shen, Bochao Wang, Hanjiang Lai, Jia Zhu, Zhiting Hu, and Jian Yin.
Towards multi-pose guided virtual try-on network.
In *Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)*, 2019a.
- Dong et al. [2019b]
Haoye Dong, Xiaodan Liang, Xiaohui Shen, Bowen Wu, Bing-Cheng Chen, and Jian Yin.
Fw-gan: Flow-navigated warping gan for video virtual try-on.
In *Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)*, 2019b.
- Dong et al. [2024]
Haoye Dong, Jun Liu, and Dong Huang.
Df-vton: Dense flow guided virtual try-on network.
In *ICASSP 2024 - 2024 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP)*, pages 3175–3179, 2024.
- Esser et al. [2021]
Patrick Esser, Robin Rombach, and Bjorn Ommer.
Taming Transformers for High-Resolution Image Synthesis.
In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*, pages 12873–12883, 2021.
- Gu et al. [2025]
Jiatao Gu, Yuyang Wang, Yizhe Zhang, Qihang Zhang, Dinghuai Zhang, Navdeep Jaitly, Josh Susskind, and Shuangfei Zhai.
Dart: Denoising autoregressive transformer for scalable text-to-image generation, 2025.
- Guo et al. [2025]
Chen Guo, Junxuan Li, Yash Kant, Yaser Sheikh, Shunsuke Saito, and Chen Cao.
Vid2avatar-pro: Authentic avatar from videos in the wild via universal prior, 2025.
- He et al. [2021]
Kaiming He, Xinlei Chen, Saining Xie, Yanghao Li, Piotr Dollár, and Ross Girshick.
Masked Autoencoders Are Scalable Vision Learners.
*arXiv:2111.06377*, 2021.
- He et al. [2022]
Kaiming He, Xinlei Chen, Saining Xie, Yanghao Li, Piotr Dollár, and Ross Girshick.
Masked Autoencoders are Scalable Vision Learners.
In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*, pages 16000–16009, 2022.
- He et al. [2024]
Kai He, Kaixin Yao, Qixuan Zhang, Jingyi Yu, Lingjie Liu, and Lan Xu.
DressCode: Autoregressively Sewing and Generating Garments from Text Guidance.
*arXiv preprint arXiv:2401.16465*, 2024.
- Ho and Salimans [2022]
Jonathan Ho and Tim Salimans.
Classifier-Free Diffusion Guidance.
*arXiv preprint arXiv:2207.12598*, 2022.
- Hu and Liu [2023]
Shoukang Hu and Ziwei Liu.
Gauhuman: Articulated gaussian splatting for real-time 3d human rendering.
*arXiv preprint*, 2023.
- Hu et al. [2025]
Shoukang Hu, Takuya Narihira, Kazumi Fukuda, Ryosuke Sawata, Takashi Shibuya, and Yuki Mitsufuji.
Humangif: Single-view human diffusion with generative prior.
*arXiv preprint arXiv:2502.12080*, 2025.
- Hu et al. [2024]
Tao Hu, Wenhang Ge, Yuyang Zhao, and Gim Hee Lee.
X-ray: A sequential 3d representation for generation.
*Advances in Neural Information Processing Systems*, 37:136193–136219, 2024.
- Jiang et al. [2024]
Boyuan Jiang, Xiaobin Hu, Donghao Luo, Qingdong He, Chengming Xu, Jinlong Peng, Jiangning Zhang, Chengjie Wang, Yunsheng Wu, and Yanwei Fu.
FitDiT: Advancing the Authentic Garment Details for High-fidelity Virtual Try-on.
*arXiv preprint arXiv:2411.10499*, 2024.
- Korosteleva and Lee [2022]
Maria Korosteleva and Sung-Hee Lee.
NeuralTailor: Reconstructing Sewing Pattern Structures from 3D Point cClouds of Garments.
*ACM Transactions on Graphics (TOG)*, 41(4):1–16, 2022.
- Korosteleva and Sorkine-Hornung [2023]
Maria Korosteleva and Olga Sorkine-Hornung.
GarmentCode: Programming Parametric Sewing Patterns.
*ACM Transactions on Graphics (TOG)*, 42(6):1–15, 2023.
- Korosteleva et al. [2024]
Maria Korosteleva, Timur Levent Kesdogan, Fabian Kemper, Stephan Wenninger, Jasmin Koller, Yuhan Zhang, Mario Botsch, and Olga Sorkine-Hornung.
GarmentCodeData: A Dataset of 3D Made-to-Measure Garments with Sewing Patterns.
*arXiv preprint arXiv:2405.17609*, 2024.
- Li et al. [2024a]
Boqian Li, Xuan Li, Ying Jiang, Tianyi Xie, Feng Gao, Huamin Wang, Yin Yang, and Chenfanfu Jiang.
GarmentDreamer: 3DGS Guided Garment Synthesis with Diverse Geometry and Texture Details.
*arXiv preprint arXiv:2405.12420*, 2024a.
- Li et al. [2025a]
Boqian Li, Haiwen Feng, Zeyu Cai, Michael J. Black, and Yuliang Xiu.
Etch: Generalizing body fitting to clothed humans via equivariant tightness, 2025a.
- Li et al. [2025b]
Tianhong Li, Yonglong Tian, He Li, Mingyang Deng, and Kaiming He.
Autoregressive Image Generation without Vector Quantization.
*Advances in Neural Information Processing Systems*, 37:56424–56445, 2025b.
- Li et al. [2024b]
Yifei Li, Hsiao-yu Chen, Egor Larionov, Nikolaos Sarafianos, Wojciech Matusik, and Tuur Stuyck.
DiffAvatar: Simulation-ready garment optimization with differentiable simulation.
In *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)*, 2024b.
- Li et al. [2025c]
Yangguang Li, Zi-Xin Zou, Zexiang Liu, Dehu Wang, Yuan Liang, Zhipeng Yu, Xingchao Liu, Yuan-Chen Guo, Ding Liang, Wanli Ouyang, et al.
Triposg: High-fidelity 3d shape synthesis using large-scale rectified flow models.
*arXiv preprint arXiv:2502.06608*, 2025c.
- Liu et al. [2023]
Lijuan Liu, Xiangyu Xu, Zhijie Lin, Jiabin Liang, and Shuicheng Yan.
Towards Garment Sewing Pattern Reconstruction from a Single Image.
*ACM Transactions on Graphics (TOG)*, 42(6):1–15, 2023.
- Liu et al. [2024a]
Yufei Liu, Junshu Tang, Chu Zheng, Shijie Zhang, Jinkun Hao, Junwei Zhu, and Dongjin Huang.
ClotheDreamer: Text-Guided Garment Generation with 3D Gaussians.
*arXiv preprint arXiv:2406.16815*, 2024a.
- Liu et al. [2024b]
Yu-Tao Liu, Xuan Gao, Weikai Chen, Jie Yang, Xiaoxu Meng, Bo Yang, and Lin Gao.
Dreamudf: Generating unsigned distance fields from a single image.
*ACM Transactions on Graphics (Proceedings of ACM SIGGRAPH 2024)*, 2024b.
- Lu et al. [2024]
Dongyue Lu, Lingdong Kong, Tianxin Huang, and Gim Hee Lee.
Geal: Generalizable 3d affordance learning with cross-modal consistency, 2024.
- Luo et al. [2024]
Zhongjin Luo, Haolin Liu, Chenghong Li, Wanghao Du, Zirong Jin, Wanhu Sun, Yinyu Nie, Weikai Chen, and Xiaoguang Han.
GarVerseLOD: High-Fidelity 3D Garment Reconstruction from a Single In-the-Wild Image using a Dataset with Levels of Details.
*ACM Transactions on Graphics (TOG)*, 43(6):1–12, 2024.
- Ma et al. [2025]
Xu Ma, Peize Sun, Haoyu Ma, Hao Tang, Chih-Yao Ma, Jialiang Wang, Kunpeng Li, Xiaoliang Dai, Yujun Shi, Xuan Ju, Yushi Hu, Artsiom Sanakoyeu, Felix Juefei-Xu, Ji Hou, Junjiao Tian, Tao Xu, Tingbo Hou, Yen-Cheng Liu, Zecheng He, Zijian He, Matt Feiszli, Peizhao Zhang, Peter Vajda, Sam Tsai, and Yun Fu.
Token-shuffle: Towards high-resolution image generation with autoregressive models, 2025.
- Mou et al. [2024]
Chong Mou, Xintao Wang, Liangbin Xie, Yanze Wu, Jian Zhang, Zhongang Qi, and Ying Shan.
T2I-Adapter: Learning Adapters to Dig Out More Controllable Ability for Text-to-Image Diffusion Models.
In *Proceedings of the AAAI conference on artificial intelligence*, pages 4296–4304, 2024.
- Nakayama et al. [2024]
Kiyohiro Nakayama, Jan Ackermann, Timur Levent Kesdogan, Yang Zheng, Maria Korosteleva, Olga Sorkine-Hornung, Leonidas J Guibas, Guandao Yang, and Gordon Wetzstein.
AIpparel: A Large Multimodal Generative Model for Digital Garments.
*arXiv preprint arXiv:2412.03937*, 2024.
- Oquab et al. [2023]
Maxime Oquab, Timothée Darcet, Théo Moutakanni, Huy Vo, Marc Szafraniec, Vasil Khalidov, Pierre Fernandez, Daniel Haziza, Francisco Massa, Alaaeldin El-Nouby, et al.
DINO2: Learning Robust Visual Features without Supervision.
*arXiv preprint arXiv:2304.07193*, 2023.
- Peebles and Xie [2023]
William Peebles and Saining Xie.
Scalable Diffusion Models with Transformers.
In *Proceedings of the IEEE/CVF International Conference on Computer Vision*, pages 4195–4205, 2023.
- Pietroni et al. [2022]
Nico Pietroni, Corentin Dumery, Raphael Falque, Mark Liu, Teresa A Vidal-Calleja, and Olga Sorkine-Hornung.
Computational Pattern Making from 3D Garment Models.
*ACM Trans. Graph.*, 41(4):157–1, 2022.
- Ravi et al. [2024]
Nikhila Ravi, Valentin Gabeur, Yuan-Ting Hu, Ronghang Hu, Chaitanya Ryali, Tengyu Ma, Haitham Khedr, Roman Rädle, Chloe Rolland, Laura Gustafson, et al.
SAM 2: Segment Anything in Images and Videos.
*arXiv preprint arXiv:2408.00714*, 2024.
- Sarafianos et al. [2024]
Nikolaos Sarafianos, Tuur Stuyck, Xiaoyu Xiang, Yilei Li, Jovan Popovic, and Rakesh Ranjan.
Garment3DGen: 3D Garment Stylization and Texture Generation.
*arXiv preprint arXiv:2403.18816*, 2024.
- Song et al. [2022]
Jiaming Song, Chenlin Meng, and Stefano Ermon.
Denoising Diffusion Implicit Models, 2022.
- Srivastava et al. [2025]
Astitva Srivastava, Pranav Manu, Amit Raj, Varun Jampani, and Avinash Sharma.
WordRobe: Text-Guided Generation of Textured 3D Garments.
In *European Conference on Computer Vision*, pages 458–475. Springer, 2025.
- Sun et al. [2024]
Peize Sun, Yi Jiang, Shoufa Chen, Shilong Zhang, Bingyue Peng, Ping Luo, and Zehuan Yuan.
Autoregressive Model Beats Diffusion: Llama for Scalable Image Generation.
*arXiv preprint arXiv:2406.06525*, 2024.
- Tang et al. [2025]
Jiaxiang Tang, Zhaoxi Chen, Xiaokang Chen, Tengfei Wang, Gang Zeng, and Ziwei Liu.
LGM: Large Multi-View Gaussian Model for High-Resolution 3D Content Creation.
In *European Conference on Computer Vision*, pages 1–18. Springer, 2025.
- Tian et al. [2024]
Keyu Tian, Yi Jiang, Zehuan Yuan, Bingyue Peng, and Liwei Wang.
Visual Autoregressive Modeling: Scalable Image Generation via Next-Scale Prediction, 2024.
- Tochilkin et al. [2024]
Dmitry Tochilkin, David Pankratz, Zexiang Liu, Zixuan Huang, Adam Letts, Yangguang Li, Ding Liang, Christian Laforte, Varun Jampani, and Yan-Pei Cao.
TripoSR: Fast 3D Object Reconstruction from a Single Image.
*arXiv preprint arXiv:2403.02151*, 2024.
- Vaswani et al. [2023]
Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N. Gomez, Lukasz Kaiser, and Illia Polosukhin.
Attention Is All You Need, 2023.
- Wang et al. [2023]
Jionghao Wang, Yuan Liu, Zhiyang Dou, Zhengming Yu, Yongqing Liang, Xin Li, Wenping Wang, Rong Xie, and Li Song.
Disentangled clothed avatar generation from text descriptions, 2023.
- Wang et al. [2025]
Rong Wang, Fabian Prada, Ziyan Wang, Zhongshi Jiang, Chengxiang Yin, Junxuan Li, Shunsuke Saito, Igor Santesteban, Javier Romero, Rohan Joshi, Hongdong Li, Jason Saragih, and Yaser Sheikh.
Fresa: Feedforward reconstruction of personalized skinned avatars from few images, 2025.
- Xiang et al. [2024]
Jianfeng Xiang, Zelong Lv, Sicheng Xu, Yu Deng, Ruicheng Wang, Bowen Zhang, Dong Chen, Xin Tong, and Jiaolong Yang.
Structured 3D Latents for Scalable and Versatile 3D Generation.
*arXiv preprint arXiv:2412.01506*, 2024.
- Xie et al. [2024]
Zhenyu Xie, Haoye Dong, Yufei Gao, Zehua Ma, and Xiaodan Liang.
Dreamvton: Customizing 3d virtual try-on with personalized diffusion models, 2024.
- Xu et al. [2024a]
Jiale Xu, Weihao Cheng, Yiming Gao, Xintao Wang, Shenghua Gao, and Ying Shan.
InstantMesh: Efficient 3D Mesh Generation from a Single Image with Sparse-View Large Reconstruction Models.
*arXiv preprint arXiv:2404.07191*, 2024a.
- Xu et al. [2024b]
Zhongcong Xu, Chaoyue Song, Guoxian Song, Jianfeng Zhang, Jun Hao Liew, Hongyi Xu, You Xie, Linjie Luo, Guosheng Lin, Jiashi Feng, and Mike Zheng Shou.
High quality human image animation using regional supervision and motion blur condition, 2024b.
- Yi et al. [2025]
Hongwei Yi, Tian Ye, Shitong Shao, Xuancheng Yang, Jiantong Zhao, Hanzhong Guo, Terrance Wang, Qingyu Yin, Zeke Xie, Lei Zhu, Wei Li, Michael Lingelbach, and Daquan Zhou.
Magicinfinite: Generating infinite talking videos with your words and voice, 2025.
- Yu et al. [2023]
Zhengming Yu, Zhiyang Dou, Xiaoxiao Long, Cheng Lin, Zekun Li, Yuan Liu, Norman Müller, Taku Komura, Marc Habermann, Christian Theobalt, et al.
Surf-d: High-quality surface generation for arbitrary topologies using diffusion models.
*arXiv preprint arXiv:2311.17050*, 2023.
- Zeng et al. [2024]
Xianfang Zeng, Xin Chen, Zhongqi Qi, Wen Liu, Zibo Zhao, Zhibin Wang, Bin Fu, Yong Liu, and Gang Yu.
Paint3D: Paint Anything 3D with Lighting-Less Texture Diffusion Models.
In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*, pages 4252–4262, 2024.
- Zhang and Wonka [2024]
Biao Zhang and Peter Wonka.
Lagem: A large geometry model for 3d representation learning and diffusion, 2024.
- Zhang et al. [2023]
Lvmin Zhang, Anyi Rao, and Maneesh Agrawala.
Adding conditional control to text-to-image diffusion models.
In *Proceedings of the IEEE/CVF International Conference on Computer Vision*, pages 3836–3847, 2023.
- Zhang et al. [2024]
Yuqing Zhang, Yuan Liu, Zhiyu Xie, Lei Yang, Zhongyuan Liu, Mengzhou Yang, Runze Zhang, Qilong Kou, Cheng Lin, Wenping Wang, et al.
DreamMat: High-quality PBR Material Generation with Geometry-and Light-aware Diffusion Models.
*ACM Transactions on Graphics (TOG)*, 43(4):1–18, 2024.
- Zhao et al. [2024]
Wang Zhao, Yan-Pei Cao, Jiale Xu, Yuejiang Dong, and Ying Shan.
DI-PCG: Diffusion-based Efficient Inverse Procedural Content Generation for High-quality 3D Asset Creation.
*arXiv preprint arXiv:2412.15200*, 2024.
- Zhu et al. [2025]
Lingting Zhu, Jingrui Ye, Runze Zhang, Zeyu Hu, Yingda Yin, Lanjiong Li, Jinnan Chen, Shengju Qian, Xin Wang, Qingmin Liao, and Lequan Yu.
Muma: 3d pbr texturing via multi-channel multi-view generation and agentic post-processing.
*arXiv preprint arXiv:2503.18461*, 2025.
- Zhuang et al. [2024]
Yiyu Zhuang, Jiaxi Lv, Hao Wen, Qing Shuai, Ailing Zeng, Hao Zhu, Shifeng Chen, Yujiu Yang, Xun Cao, and Wei Liu.
Idol: Instant photorealistic 3d human creation from a single image, 2024.

## 6 More Implementation Details

The MAR component adopts a Transformer-based encoder-decoder architecture, featuring 16 layers in both the encoder and decoder paths, 16 attention heads per layer, and an embedding dimension of 768. This design choice facilitates effective modeling of long-range dependencies in 3D geometry. The MLP-based diffusion module consists of 6 layers with a width of 1024, which provides sufficient capacity for the denoising process while maintaining computational efficiency. We train our model using the AdamW optimizer with an initial learning rate of $1e-4$, ${\beta_{1}}$=0.9, ${\beta_{2}}$=0.999, $\varepsilon=1\times{10^{-8}}$, and a weight decay of $0.01$. The learning rate follows a cosine annealing schedule with warm restarts, starting with an initial cycle length of 10 epochs and decaying to 1% of the initial rate. This scheduling strategy helps stabilize early training while ensuring convergence in later stages. During inference, the reverse denoising steps are set to 100, and the auto-regressive steps are set to 32 for MAR.

## 7 More Qualitative Results

Here we show more qualitative results of GarmentX. As shown in the first three rows of Figure [12](https://arxiv.org/html/2504.20409v1#S9.F12), our framework successfully generates various types of lower garments, accurately distinguishing between full-length pants (a, b, c, d), shorts (e), miniskirts (f, g), and bodycon skirts (h, i). The subsequent rows showcase generated dresses and upper garments, our method captures intricate geometric details including fitted cuffs (j, k), collars (l), sleeveless design (l, m, n, o), and pleated skirt design (o).

## 8 Future Works

The current GarmentX implementation processes only human-free garment images due to our automatic data generation pipeline’s human-free design. For clothed human inputs as shown in Figure [10](https://arxiv.org/html/2504.20409v1#S9.F10), we employ SAM2 to segment for preprocessing. Planned improvements include expanding the automatic data construction pipeline to encompass clothed-human images, thereby enabling end-to-end processing of clothed-human images.

## 9 Failure Cases

While GarmentX supports a wide range of modern garment categories, Figure [11](https://arxiv.org/html/2504.20409v1#S9.F11) reveals failure cases in processing highly complex designs—particularly those containing complex decorations, like oversized chest-attached bow knots and petal-like designs. For these challenging cases where even professional artists would struggle to conceptualize corresponding sewing patterns, our system nevertheless generates the closest valid parameter set and sewing patterns to approximate the images. This contrasts with direct sewing pattern prediction methods that often yield simulating failed patterns under such complexity.

Figure: Figure 10: Segmentation result as Input. We use SAM2 to get the segmentation result and use it as the input to our model.
Refer to caption: extracted/6397419/imgs/sam_result.png

Figure: Figure 11: Some failure cases. GarmentX fails when handling highly complex cases.
Refer to caption: extracted/6397419/imgs/failed_case.png

Figure: Figure 12: More qualitative results. GarmentX can handle diverse input images with various styles and textures. It accurately captures geometric details in the input images and generates high fidelity, open-structure and simulation-ready 3D garments, facilitating downstream applications.
Refer to caption: extracted/6397419/imgs/more_result.png