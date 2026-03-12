<!-- markdownlint-disable -->
## Contents
- 1. Introduction
- 2. Related work
  - Garment shape design.
  - Sewing pattern design and optimization.
  - Patch layout decomposition.
  - Surface parameterization and textile distortion measures.
- 3. Overview
  - 3.1. Requirements
  - 3.2. Pipeline structure
- 4. Method
  - 4.1. Cross-field construction
  - 4.2. Patch layout creation
    - Path tracing
    - Path classification.
    - Path insertion.
    - Path removal
    - Dart creation.
    - Symmetry.
  - 4.3. Anisotropic textile parameterization
    - Stretch.
    - Shear and rigidity.
    - Seam reflection symmetry.
    - Dart symmetry.
    - Grain alignment.
    - Optimization.
- 5. Results
- 6. Conclusions
  - Limitations and future work.
- References

## Abstract

Abstract. We propose a method for computing a sewing pattern of a given 3D garment model. Our algorithm segments an input 3D garment shape into patches and computes their 2D parameterization, resulting in pattern pieces that can be cut out of fabric and sewn together to manufacture the garment. Unlike the general state-of-the-art approaches for surface cutting and flattening, our method explicitly targets garment fabrication. It accounts for the unique properties and constraints of tailoring, such as seam symmetry, the usage of darts, fabric grain alignment, and a flattening distortion measure that models woven fabric deformation, respecting its anisotropic behavior. We bootstrap a recent patch layout approach developed for quadrilateral remeshing and adapt it to the purpose of computational pattern making, ensuring that the deformation of each pattern piece stays within prescribed bounds of cloth stress. While our algorithm can automatically produce the sewing patterns, it is fast enough to admit user input to creatively iterate on the pattern design. Our method can take several target poses of the 3D garment into account and integrate them into the sewing pattern design. We demonstrate results on both skintight and loose garments, showcasing the versatile application possibilities of our approach.

## 1. Introduction

In this work, we propose a method for automatically creating a sewing pattern for a given 3D model of a garment.

In the fashion industry, the garment creation process starts from the 2D domain: The pattern maker creates the 2D sewing pattern using traditional, often tacit knowledge (Chen, 1998), established templates and a few standard measurements, such as waist circumference, shoulder width, etc. The cut fabric pieces are then sewn together to form the garment. Designers may work with mannequins to experiment with the desired shape and draping of the fabric in the physical 3D space, but the ultimate determination of the garment shape comes from the 2D pattern. Also, in digital garment design tools such as Clo3D (2022), the 2D pattern is required to simulate and drape the garment on a virtual mannequin or avatar, and thus the design process is centered around the sewing pattern. Pattern making is currently a manual, time-consuming task requiring much experience and skill. One of the consequences is that custom made clothing that perfectly fits the intended wearer is a luxury available to the privileged few. The majority of clothing is mass-produced using standard sizes based on averaged measurements that do not fit well for most people (SizeGermany, 2020).

There are many powerful techniques to model shapes directly in 3D, with various approaches specifically developed for 3D garment modeling, e.g. with sketch based user interfaces (Wolff et al., 2021; Rose
et al., 2007; Decaudin et al., 2006; Turquin et al., 2007; Robson
et al., 2011), input gestures in the physical (Wibowo
et al., 2012) or virtual reality (TiltBrush, 2022), via 3D scanning (Pons-Moll
et al., 2017) or data driven modeling (Wang
et al., 2018). Advances in human body modeling (Bogo
et al., 2014; Loper et al., 2015; Osman
et al., 2020; Alldieck
et al., 2021) facilitate custom digital tailoring of garments that fit the personalized body avatar without having to rely on standard sizes. However, the 2D sewing pattern of the garment is still necessary for accurate cloth simulation and the manufacturing of the physical garment.

To create the 2D pattern, the 3D garment surface is divided into patches, also called panels, and each panel is flattened onto the 2D domain. Many methods exist for surface segmentation and low-distortion parameterization, as we discuss in Sec. [2](#S2). State-of-the-art approaches are able to compute optimal cuts that balance the flattening distortion and cut length (Sharp and Crane, 2018; Li
et al., 2018a; Poranne et al., 2017). Recent work (Wolff et al., 2021) applies such a technique to create the sewing pattern, but the resulting panel shapes are visually far from common practice in fashion and challenging to sew. The problem is that most segmentation techniques are general and do not account for the specific setting of cloth pattern making and the fabrication constraints unique to it. In particular, parameterizing a 3D shape to be made of woven textile requires a special deformation model aware of the thread properties and fabric grain alignment, as well as shape and length constraints on the seams and garment aesthetics. For example, to sew together two pieces of cloth, in practice, the two matching seams must be of the same length, as straight or at least smooth as possible, and ideally they should be reflection-symmetric in 2D, so that the tailor can put the panels one on top of the other and sew them together along one planar curve.

In this work, we propose a shape segmentation and patch flattening method specifically intended for pattern making (see Fig. [1](#S0.F1)). Our core contributions are a 3D patch layout creation approach informed by a geometric measure of woven fabric distortion and tailoring fabrication requirements, as well as a patch parameterization method that minimizes the textile deformation measure and ensures that the pattern pieces are sewable. Our method incorporates mechanisms of classical pattern making, such as the usage of darts, preference towards global symmetry and vertical grain alignment by default. Our algorithm can work fully automatically, but it is efficient enough to admit interactive input, allowing creative design iteration in real time. The designer can sketch on the 3D garment model to hint at desired seams and textile grain alignment and vary the flattening parameters according to the physical properties of the desired fabric. As an additional option, if multiple poses of the garment corresponding to various body poses are available, our algorithm can adapt the pattern by integrating the information from all the poses. We demonstrate results on a number of tight-fitting and loose garments, showcasing the usability and versatility of our method. To foster future research on digital fashion, we will publicly release our software implementation.

## 2. Related work

Digital garment design and fabrication pose several fundamental, interconnected research problems in geometry processing and physics based simulation. We give a brief review of the methods for surface modeling, patch decomposition and flattening in the context of computational pattern making.

#### Garment shape design.

The digitalization of the fashion industry in general and garment design in particular bears multiple economical, ecological and societal advantages and poses fascinating research challenges, sparking significant interest (Nayak and Padhye, 2017).
Clo3D (2022) and Optitex (2022) are examples of common CAD tools used in the industry. Such interactive CAD editors allow the designer to create sewing patterns in 2D and simulate their physical appearance and draping on an avatar in 3D. Adjusting the 3D garment shape requires changing the 2D pattern via a trial-and-error process. In the research community, Umetani et al. (2011) propose bidirectional interactive garment editing, leveraging fast cloth simulation to enable users to work in 2D and 3D simultaneously and observe the effects of changes in both modes.

Figure: Figure 2. Top: matching seams on two patches to be stitched together, as well as the sides of a dart must be of equal length and ideally reflection-symmetric. Bottom: the fabric grain of pattern pieces aims to align with the vertical direction on the worn garment.
Refer to caption: /html/2202.10272/assets/img/fig2/reflec_illus.png

Figure: Figure 3. An overview of the different steps of our pipeline. The shape is first symmetrized (a), a smooth cross-field is computed on the mesh (b), paths are then traced on the shape by following the cross-field (c), and the resulting patches are then flattened onto the $UV$ plane to create a 2D sewing pattern (d).
Refer to caption: /html/2202.10272/assets/img/pipeline/0_symm_0.png

Various computational approaches create 3D garment shapes from contours, seam-, fold- and boundary curves, sketched by the user on an avatar, see e.g. (Turquin et al., 2007; Decaudin et al., 2006; Rose
et al., 2007; Robson
et al., 2011; Wolff et al., 2021).
Other approaches leverage data and neural networks to construct 3D garments from 3D scans (Chen
et al., 2015; Pons-Moll
et al., 2017; Bang
et al., 2021) or by using parametric models and learning (Wang
et al., 2018; Vidaurre et al., 2020); a recent data set of 3D garments with corresponding 2D patterns can be used for supervision (Korosteleva and
Lee, 2021). To simulate and further process the garment models, most methods require 2D patterns, which are either provided as input or computed using surface segmentation and parameterization, as discussed below. Dedicated approaches to knitwear design and fabrication are explored in (McCann et al., 2016; Narayanan* et al., 2019; Yuksel
et al., 2012; Narayanan et al., 2018; Wu
et al., 2019), where a sewing pattern is not needed, since knitting machines can continuously knit various topologies. By contrast, in this paper we focus on pattern based garment construction from woven fabric (or knitted fabric such as jersey, treated similarly), which is an overwhelmingly widespread practice in the garment industry.

#### Sewing pattern design and optimization.

Several methods *modify given* garment patterns to fit a particular body shape (Cordier
et al., 2003; Wang
et al., 2005b; Brouet
et al., 2012; Meng
et al., 2012; Bartle et al., 2016; Wang, 2018; Bang
et al., 2021; Liu et al., 2018), optimize stress, pressure and seam traction (Montes
et al., 2020) or generate user-defined target folds (Li
et al., 2018b). They do not modify the given topological patch layout. In contrast, our method *generates* a sewing pattern directly from the 3D garment model while respecting fabric stress bounds and fabrication constraints.

The evolutionary algorithm by Kwok et al. (2015) generates a sewing pattern from 3D body geometry, but it does not account for fit or fabric stress and manufacturing requirements.

#### Patch layout decomposition.

General surface segmentation methods for the purpose of low-distortion flattening aim at creating a small number of patches (or short cuts) to reduce Gaussian curvature. A number of works compute the patches and cuts jointly with the parameterization in order to optimize or bound both (Sorkine et al., 2002; Poranne et al., 2017; Li
et al., 2018a). Variational surface cutting (Sharp and Crane, 2018) optimizes the tradeoff between a given distortion measure and cut length. Another class of works computes approximations of a given shape with (nearly) developable surfaces (Julius
et al., 2005; Stein
et al., 2018; Ion et al., 2020; Binninger* et al., 2021). However, the seams generated by all such methods are not appropriate for garment fabrication because they are far too complex and difficult to sew and do not necessarily fulfill even the basic requirement of equal length for matching seams, nor symmetry.
Huang et al. (2012) use predefined cuts based on an anthropomorphic subdivision, which do not generalize to other garments or body shapes.

Methods for patch layout generation for the purpose of globally smooth parameterization and semiregular remeshing focus on the quality of the layout and its combinatorial structure (Campen, 2017). State-of-the-art techniques compute a smooth tangent vector field on the surface (Vaxman et al., 2017) aligned to features and principal directions as the basis for tracing the patch decomposition (Razafindrazaka et al., 2015; Pietroni et al., 2016; Nuvoli et al., 2019; Livesu et al., 2020; Pietroni et al., 2021). The topology of these layouts is restricted by the remeshing application, e.g. four corners per patch for quadrangulation, which is too stringent for pattern making. We adapt the method by Pietroni et al. (2021) to suit the different setting of garment design and fabrication, where the topology requirements are different and textile based distortion measures must be bounded.

#### Surface parameterization and textile distortion measures.

General flattening methods take a surface of disk topology and find its mapping to 2D that minimizes a distortion measure (Hormann
et al., 2007). The used objective most typically measures conformal distortion (e.g. (Lévy et al., 2002; Sheffer et al., 2004)) or deviation from isometry (e.g. (Liu
et al., 2008; Rabinovich et al., 2017)). Guaranteeing locally and globally injective parameterization is possible (Sawhney and Crane, 2017; Jiang
et al., 2017) but can be computationally costly. Most importantly, the general distortion measures are isotropic and do not account for the particular behavior of woven fabric, which is almost inextensible along the yarn directions and more stretchable in the diagonal direction. In other words, developable surfaces are global minimizers of all these distortion measures, but non-vanishing locally minimal energy states do not necessarily model plausible fabric configurations.

McCartney et al. (2000; 2005) and Wang et al. (2005a) look specifically at models for woven fabric and flatten the mesh by discriminatively optimizing the yarn stretch and shear. Wang et al. (2005a) create a discrete orthogonal grid to represent the textile and wrap it around a target 3D mesh, thereby establishing the 2D-to-3D mapping. McCartney et al. opt for a continuous approach, where the intextensible yarn directions are represented by the UV-axes in the flat domain. Their energy optimization is nonlinear and its optimization could be impractical for realtime interactive design iteration framework. Our textile deformation model is inspired by these works, but we use a different optimization strategy to achieve fast performance, and we incorporate seam symmetry (Wolff
et al., 2019) and grain alignment constraints for fabrication feasibility.

## 3. Overview

Our pipeline takes as input a triangle mesh representing the target garment in 3D and automatically produces a 2D sewing pattern. The designer can influence the final layout by sketching some curves on the surface, indicating desired seams. In addition to the rest shape in 3D, our approach can also consider additional target poses that the garment should assume when worn.

Our method is intended for woven fabrics that consist of warp and weft yarns. The warp direction is the main (longitudinal) direction of the fabric, also termed *grain*, and the weft direction is orthogonal to it. Depending on the material, the textile threads can be elastic or nearly inextensible, determining the stretching resistance of the fabric along the warp and weft directions. The fabric can usually stretch more along the diagonal direction, resulting in shear of the woven structure. The main parameters of our framework are therefore the maximal admissible stretch and shear, determining the maximum deformation that the material undergoes from the flat configuration to the draped 3D shape. Note that excessive shear is undesirable, even if physically possible, because it leads to wrinkles and impression of bad fit. Ideally, the deformation of the pattern pieces is close to isometry. We assume the fabric is sufficiently thin and its resistance to bending too small influence the pattern making.

### 3.1. Requirements

To compute a usable, manufacturable sewing pattern for a garment, our algorithm should fulfill several (interdependent) requirements.

*Patch shape.* The pattern should preferably consist of few large patches with straight or at least smooth boundary segments. Practical patterns pieces have few corners (between 6 and 8), and their angles should be approximately orthogonal. If a patch contains darts, their opening angles should also be sufficiently large, and the dart length usually should not exceed a fraction of the patch length, as it is then easier in practice to cut through and sew two separate pieces together.

*Bounded fabric strain.* According to the mentioned fabric parameters, the textile cannot exceed the prescribed deformation threshold when draped into the target 3D shape. The lower this bound, the more pattern pieces are required for doubly curved target shapes.

*Seam sewing feasibility.* Matching seams to be sewn together must have equal length. In practice, straight seams are easiest to sew, and it is desirable that seams can be sewn flat, meaning that the matching seams must be reflection-symmetric, so that the two pieces of fabric can be placed on top of each other to sew (Fig. [2](#S2.F2), top).

*Layout symmetry.* Aesthetics is strongly associated with symmetry, and therefore symmetric sewing patterns are desirable, in particular when the targeted garment design is symmetric.

*Grain alignment.* A pattern piece should be oriented in 2D such that the fabric grain is roughly aligned with the vertical (gravity) direction when the garment is worn, or with some other axis, such as along the arm (see Fig. [2](#S2.F2), bottom). Grain alignment ensures predictable, symmetric behavior when the garment is draped and subject to gravity, and pressure of the body, as well as when shrinkage due to washing occurs. Sometimes the designer may wish to prescribe a special grain direction, as in bias cut, where the grain is at 45 degrees to the vertical direction, e.g. in some skirt designs.

*Efficiency.* Constructing an efficient pipeline allows the designer to creatively iterate and adjust the patch layout by providing simple input by interactively sketching on the 3D surface.

### 3.2. Pipeline structure

Fig. [3](#S2.F3) summarizes our processing pipeline.

*Input.* We assume the input garment mesh to be a manifold and with well shaped triangles (otherwise we uniformly remesh with standard tools).
If additional poses of the 3D garment are available, we assume their meshes share the connectivity and are in full correspondence with the rest pose.
If the input garment is symmetric, we can enforce symmetry throughout the pipeline by splitting the mesh along the symmetry plane, running the method on one side, and then reflecting the result (Fig. [3](#S2.F3)a).

*Cross-field construction.* We compute a smooth 4-rotational-symmetric (4-RoSy) tangent vector field on the surface (Fig. [3](#S2.F3)b) aligned with principal curvature directions and boundaries. If the input includes multiple poses, we integrate the contribution of the curvature of all the poses in the derived field (see Sec. [4.1](#S4.SS1)).

*Layout construction.* With the goal of producing pattern pieces that fulfill the requirements above, we trace a set of *paths* across the mesh to partition the surface into the different panels (see Sec. [4.2](#S4.SS2) and Fig. [3](#S2.F3)c). The paths are oriented to follow the underlying cross-field and include the curves sketched by the designer. The patch formation is controlled by the textile distortion measure, and the designer can specify the maximum number of corners allowed in a patch, to control patch complexity. At the end of this step, the seam on the symmetry plane can be optionally removed, if possible.

*Patch flattening.*
Finally, each patch is flattened onto the 2D space and packed with the others into a single textile sheet (see Fig. [3](#S2.F3)d). This step is achieved by a novel parameterization method that mimics textile physics using geometric measures.
We impose constraints on the seams to ensure that they can be physically sewn. The parameterization step is also deployed during patch layout construction to individually uphold the distortion threshold of each patch.

## 4. Method

The following sections detail each phase of our processing pipeline.

### 4.1. Cross-field construction

Figure: Figure 4. Cross-field computation: The multi-scale main curvature directions used as soft constraint to retrieve a smooth field.
Refer to caption: /html/2202.10272/assets/img/field/field_curvature.png

Given a proper input, we initially construct a cross-field (a 4-RoSy tangent-vector field (Vaxman et al., 2017)) on the input surface. The cross-field is crucial in our framework as it drives the entire tracing process; the cuts follow the directions expressed by the cross-field. Intuitively, aligning the seams to the main curvature directions is an excellent strategy to maximize the developability of the resulting patches. A significant collection of quadrangulation methods have demonstrated the strong correlation that exists between developability and curvature alignment (Bommes
et al., 2009; Bommes et al., 2013; Pietroni et al., 2021).

We initialize the field by extracting curvature directions at a low scale using the method proposed in (Panozzo
et al., 2010). Similarly to (Bommes
et al., 2009) we use the anisotropy of the curvature directions
to detect the regions where these curvatures are essential (see Fig. [4](#S4.F4), left).

Then we run the globally smooth method proposed by Diamanti et al. (2014), adding soft constraints to align the field to the principal curvature directions, similarly to (Panozzo
et al., 2012). The anisotropy weighs each soft constraint value we previously computed to adhere to the principal curvature directions where those are relevant, and smooth on the rest. In addition to these soft constraints, we impose the field to align simultaneously with the user-defined constraints and the boundary edges of the 3D mesh. This way, the path will hit the boundary orthogonally, and we implicitly avoid creating artifacts or long stripes (Fig. [9](#S4.F9)).
The resulting field is shown in Fig. [4](#S4.F4), right.

When multiple frames are provided, we first compute the multiscale curvature and the anisotropy for each frame. Then, we average the field directions for each face in the rest shape (we transport the field on the best shape and perform $\pi/2$ invariant interpolation of the cross-field, weighted by the anisotropy values). Lastly, we smooth all the fields together in the rest position and average the anisotropy among all frames.

### 4.2. Patch layout creation

This pipeline step obtains a patch layout as the byproduct of an iterative process that traces field-aligned paths over the target surface. The network of paths designs a patch layout composed of rectangular or non-rectangular patches. Distinct paths can only intersect orthogonally on the surface. Two intersecting paths can cross or stop when they meet, forming a T-junction. We aim at inserting the optimal number of paths in the right locations to obtain patches that satisfy the requirements we introduced in Sec. [3](#S3).

#### Path tracing

Figure: Figure 5. The construction of the tracing graph. Each mesh vertex spawns 4 nodes in the graph, corresponding to the 4 components of the cross-field. Edges in the graph connect nodes of adjacent vertices that represent matching cross-field directions.
Refer to caption: /html/2202.10272/assets/img/graph/M4.png

To trace field-oriented paths, we use the graph-based approach proposed by Nuvoli et al. (2019) and Pietroni et al. (2021). First, we interpolate the cross-field on vertices (considering their invariance to 90-degree rotations). We create a graph having four nodes for each mesh vertex, one for each direction of its cross-field. Then, we connect adjacent nodes with matching cross-field direction.
Finally, we associate to each connection of the graph a weight that depends on how much it drifts from the direction defined at the adjacent nodes (see Fig. [5](#S4.F5)). For details on the graph creation we refer to (Pietroni et al., 2021, 2016).

#### Path classification.

Figure: Figure 6. Loops and border-to-border paths are both useful in most of the cases to obtain a proper patch decomposition.
Refer to caption: /html/2202.10272/assets/img/loop_vs_border/loop_vs_border.png

We trace two classes of paths: paths connecting border-to-border and paths connecting to themselves, i.e., loops. We found that both categories of these paths are fundamental to assembling a proper patch layout (see Fig. [6](#S4.F6)). We formulate the problem of path tracing as a shortest-path search between a given *source node* and a set of potential *destination nodes* in the graph. In particular, when tracing a *loop* from a specific vertex, we select an internal node as the source and we aim at coming back to the same node (see Fig. [7](#S4.F7), left).

Figure: Figure 7. Loop tracing (left) and border-to-border tracing (right).
Refer to caption: /html/2202.10272/assets/img/tracing_type.png

Figure: Figure 8. Loops and border-to-border paths are inserted iteratively until the goals for each patch are satisfied. In the removal step, patches are fused by removing paths if the fused patch still satisfies our goals.
Refer to caption: /html/2202.10272/assets/img/sampling/sample0.png

Figure: Figure 9. The insertion of border-to-border paths that would generate long thin strips, such as the one shown in blue, is avoided by selecting vertices that *enter* and *exit* the mesh w.r.t. the cross-field.
Refer to caption: /html/2202.10272/assets/img/sleeves/sleeves.png

Similarly, for border-to-border paths, we select as source a node lying on a boundary vertex whose direction *enters* the mesh, and as a destination all the border nodes whose directions *exit* the mesh (see Fig. [7](#S4.F7), right). This trick avoids any path to form strange, long strips when touching the border (see Fig. [9](#S4.F9)). In a preprocessing step, we mark each boundary node with a label to determine if it is an exit or an entrance node. Since we align the field to boundaries, we always have these two kinds of nodes for each boundary vertex.

Paths are sequences of adjacent edges that belong to the triangle mesh. So, they usually have irregular shapes. We smooth the paths and reproject them over the surface to make them more regular and we update the rest of the mesh as a consequence.

#### Path insertion.

We initially sample a set of candidate paths by tracing border-to-border paths and loops. For the sake of efficiency, we subsample uniformly the source nodes on the border and in the interior of the mesh. Similarly to (Livesu et al., 2020), we insert a path using a greedy strategy that favors the furthest path from the previously inserted one.
As in (Pietroni et al., 2021, 2016), the distance is computed for each node using the M4 stratification of the graph (Campen
et al., 2012), and averaged for each path. Notice that two paths crossing orthogonally can be very far. In contrast, parallel paths tend to be close. This simple strategy avoids conglomerations of paths.

We keep the complete patch layout updated at every insertion step, and we mark all the patches that do not satisfy the goals specified in Sec. [3.1](#S3.SS1). If a candidate path splits one patch that does not meet our goals and does not intersect tangentially with any previously inserted path, then we insert it. As in (Pietroni et al., 2016), given two candidate paths, it is trivial to check whether they intersect tangentially by simply testing whether they pass the same vertex through non-orthogonal directions.

We repeat this insertion strategy until all the patches match our goals. We perform a recursive step on each patch if needed. Notice that this step needs new patches to be parameterized at every insertion step to check whether the produced mapping fulfills the bijectivity and bounded distortion requirements, hence the need for a fast and reliable method to parameterize and test the produced patches. We illustrate the steps of this sampling procedure in Fig. [8](#S4.F8), left.

#### Path removal

Once the path insertion is complete, we have a patch layout satisfying our goals. However, during its construction, there is no easy way to predict whether a candidate path is essential in the final layout in its entirety (since some goals can only be satisfied by a combination of multiple paths). We remove redundant path segments starting from the last inserted path in reverse order.
Fig. [8](#S4.F8) (right) shows the effect of the removal procedure. In this step, we disable the removal of path segments that form T-junctions to avoid creating darts, as these are treated explicitly in the next step.

#### Dart creation.

We removed all the redundant path segments at the end of the previous step. However, there might be adjacent patches that we can *partially* glue together, maintaining the distortion under the predefined threshold. Such partial cuts can be found in fashion design and are usually referred to as darts. In traditional pattern making, a tailor introduces darts to better shape the body’s curves. By removing a wedge-shaped piece of fabric and sewing both sides together, an experienced tailor effectively introduces angle deficiency on the garment and thus creates a curved shape from a flat pattern.

Figure: Figure 10. Creating darts starting from areas with negative curvature tends to generate overlaps in the $UV$ mapping.
Refer to caption: /html/2202.10272/assets/img/wrong_dart/wrong_darts0.png

To introduce darts, we first split all paths into their segments, at the intersections with the other paths. Then we sort the different path segments by considering the Gaussian curvature of the mesh region they span (as in field computation, we extract curvature at a low scale using the method proposed by Panozzo
et al. (2010)). Ideally, we want to merge starting from the regions with lower Gaussian curvature, as such seams are more likely to be merged with low distortion. Similarly, areas with negative Gaussian curvature (saddles) should be merged later, as they most likely generate overlaps in the $UV$ domain (see Fig. [10](#S4.F10)).
Then we select the first path segment and split it uniformly into a number of subpaths. Following the same intuition, we start merging from the lower curvature side, and we continue as long as the produced distortion is below the threshold.

#### Symmetry.

Figure: Figure 11. The seam on the global symmetry plane can be safely removed at the end of the layout generation process.
Refer to caption: /html/2202.10272/assets/img/symmetry_removal/symm_rem_0.png

The generation of the patch layout is conducted on one side of the mesh using a symmetry plane (shown in red in Fig. [3](#S2.F3)a). We first copy the patch layout on the other symmetric side, then we start the path removal and dart insertion process for the paths laying on the symmetry plane. This way, we keep the symmetric distribution and, at the same time, remove unwanted seams along the symmetry plane (see Fig. [11](#S4.F11)).

### 4.3. Anisotropic textile parameterization

Woven fabric can be modeled as a regular grid of threads, where the two axes are called warp and weft (or collectively ‘grain’). We assume the common case where the grid is orthogonal, and thus represent it by the $UV$ axes in the flat domain, but it is possible to model arbitrary angles between the warp and the weft directions (McCartney
et al., 2000, 2005). Ideally, the cloth undergoes only bending when the flat pattern is draped on the 3D body, but this implies that the 3D garment shape is piecewise developable, which is not always possible. We hence need to measure deviation from developability in a way that is consistent with the woven structure properties. General parameterization distortion measures are isotropic, or invariant to rotations, but this ignores the fact that the threads are nearly inextensible, while a certain, limited angular distortion is permitted, which means that the cloth can stretch diagonally to the warp and weft ($UV$) directions, but much less so along those directions. Hence the intrinsic parameterization distortion measure needs to be anisotropic. We penalize stretch along the $U$ and $V$ directions separately from shear. Note that by shear we mean angular distortion of the fabric grid, while the thread lengths are preserved, so it is not an area-preserving shear transformation (see Fig. [12](#S4.F12)).

Figure: Figure 12. Woven net (left) undergoing stretch (middle) and shear (right).
Refer to caption: /html/2202.10272/assets/img/stretch/stretch_shear_viz2.png

Figure: Figure 13. Per-triangle flattening, modeled as an affine map $f$.
Refer to caption: /html/2202.10272/assets/img/triangle_viz.png

Consider a reference triangle $A^{\prime}B^{\prime}C^{\prime}$ in $XYZ$ coordinates mapped to a distorted triangle $ABC$ in $uv$ coordinates, see Fig. [13](#S4.F13). A unique affine transformation $f$ maps $A^{\prime}B^{\prime}C^{\prime}$ to $ABC$.
To penalize stretch of the fabric grain, we can consider the scaling induced by the inverse transformation $f^{-1}:\mathds{R}^{2}\rightarrow\mathds{R}^{3}$, which can be interpreted as the stretch of a canonical unit frame $\mathcal{G}$ in the $UV$ plane,
undergoing the transformation $f^{-1}$, as illustrated in Fig. [13](#S4.F13). The stretch values $s_{u}$ and $s_{v}$ can be expressed using the columns of the Jacobian of $f^{-1}$ (i.e., the tangents of the parameterization), denoted $J=[J_{1}\ J_{2}]\in\mathds{R}^{3\times 2}$:

$$ (1) $\displaystyle s_{u}=\|J_{1}\|\text{ , }s_{v}=\|J_{2}\|.$ $$

#### Stretch.

We could define the stretch penalty as the deviations of $s_{u},s_{v}$ from $1$. However, these scaling factors are non-linear functions of the variables in our problem, namely the $UV$ coordinates of the triangle vertices $ABC$, leading to a non-quadratic stretch measure. We therefore invert the problem and consider the stretching of the *image* of frame $\mathcal{G}$, namely $J(\mathcal{G})$, when $f$ is applied.

For simplicity, we express the frame $\mathcal{G}$ explicitly by three points: the 2D triangle centroid $G$, and the points $G_{u}=G+(1,0),\ G_{v}=G+(0,1)$ (see Fig. [13](#S4.F13)).
The 3D grain directions $J(\mathcal{G})$ can be expressed by transforming these three points by $f$ into $G^{\prime},G^{\prime}_{u},G^{\prime}_{v}$. We denote by $(\alpha_{X},\beta_{X},\gamma_{X})$ the barycentric coordinates of any point $X$ w.r.t. triangle $ABC$. Then we have:

$$ (2) $\displaystyle G^{\prime}=\alpha_{G}A^{\prime}+\beta_{G}B^{\prime}+\gamma_{G}C^{\prime},$ (3) $\displaystyle G^{\prime}_{u}=\alpha_{G_{u}}A^{\prime}+\beta_{G_{u}}B^{\prime}+\gamma_{G_{u}}C^{\prime},$ (4) $\displaystyle G^{\prime}_{v}=\alpha_{G_{v}}A^{\prime}+\beta_{G_{v}}B^{\prime}+\gamma_{G_{v}}C^{\prime}.$ $$

This allows us to measure the lengths $s_{u}=\|G^{\prime}_{u}-G^{\prime}\|$ and $s_{v}=\|G^{\prime}_{v}-G^{\prime}\|$ on the 3D triangle. If the mapping $f$ does not stretch the $J_{1}$ direction, then $\|G_{u}-G\|=s_{u}$, and equivalently for $J_{2}$ and $v$. We denote the $UV$ coordinates of our helper points as $G=(u_{G},v_{G}),G_{u}=(u_{G_{u}},v_{G_{u}}),G_{v}=(u_{G_{v}},v_{G_{v}})$, and then the stretch of $J_{1}$ caused by $f$ is $\|G_{u}-G\|$, which is equal to
$|u_{G_{u}}-u_{G}|=(\alpha_{G_{u}}-\alpha_{G})u_{A}+(\beta_{G_{u}}-\beta_{G})u_{B}+(\gamma_{G_{u}}-\gamma_{G})u_{C},$
and similarly for the other direction.
We thus define the per-triangle stretch energy terms as follows:

$$ (5) $\displaystyle E_{\mathrm{stretch},u}(ABC)=$ $\displaystyle\phantom{a}\omega_{\mathrm{stretch}}\left[s_{u}-\left((\alpha_{G_{u}}-\alpha_{G})u_{A}+(\beta_{G_{u}}-\beta_{G})u_{B}+(\gamma_{G_{u}}-\gamma_{G})u_{C}\right)\right]^{2},$ $\displaystyle E_{\mathrm{stretch},v}(ABC)=$ $\displaystyle\phantom{a}\omega_{\mathrm{stretch}}\left[s_{v}-\left((\alpha_{G_{v}}-\alpha_{G})v_{A}+(\beta_{G_{v}}-\beta_{G})v_{B}+(\gamma_{G_{v}}-\gamma_{G})v_{C}\right)\right]^{2}.$ $$

The total energy $E_{\mathrm{stretch},u}$ is then defined as the sum of per-triangle energies, and similarly for $E_{\mathrm{stretch},v}$.

#### Shear and rigidity.

A direct expression for shear is the angle between the parameterization tangents $J_{1}$ and $J_{2}$, with the constraint that the tangents maintain their length, but this is again not a quadratic in the variables $u_{A},v_{A},\ldots$ Instead, we propose to measure deviation from isometry, implicitly penalizing shear in the same manner as in as-rigid-as-possible (ARAP) parameterization (Liu
et al., 2008):

$$ (6) $\displaystyle E_{\mathrm{rigid}}(ABC)=\omega_{\mathrm{rigid}}\sum_{e\in ABC}(u_{e}-u_{e^{\prime}})^{2}+(v_{e}-v_{e^{\prime}})^{2},$ $$

where $e$ denotes a triangle edge in triangle $ABC$, $e^{\prime}$ is the corresponding edge in triangle $M(A^{\prime}B^{\prime}C^{\prime})$ and $M$ is the best-fit rigid transformation that aligns $A^{\prime}B^{\prime}C^{\prime}$ to $ABC$, computed using Procrustes (Sorkine-Hornung and Rabinovich, 2016).
The total rigidity measure $E_{\mathrm{rigid}}$ sums up Eq. ([6](#S4.E6)) over all triangles.
Although this ARAP measure mixes shear with isotropic stretch, it tends to produce well conditioned results thanks to the Procrustean step and works well for our purpose.

#### Seam reflection symmetry.

Pattern pieces are sewn together along seams by placing one part onto the corresponding part and stitching along a curve. This implies the existence of a perfect reflection between the matching borders that get sewn together. We represented matching seams in the $UV$ domain as two sets of duplicated vertices $\mathcal{P}=(p_{1},\ldots,p_{n})$ and $\mathcal{Q}=(q_{1},\ldots,q_{n})$, where $p_{i}$ and $q_{i}$ correspond to the same 3D location but are mapped to different locations in 2D.

To measure the deviation of the matching seams from a perfect reflection, we first compute the best-fit reflection transformation $M$ between $\mathcal{P}$ and $\mathcal{Q}$, similarly to Wolff et al.  (2019). We achieve this by simply switching the sign of the determinant during procrustean analysis.

If the $UV$ seams are reflection-symmetric, $M\mathcal{P}$ and $\mathcal{Q}$ coincide, as well as $\mathcal{P}$ and $M^{-1}\mathcal{Q}$. Otherwise, we define target points as:

$$ (7) $\forall i\in[1\ldots n],\ \ q_{t,i}=\frac{Mp_{i}+q_{i}}{2}\text{ , }p_{t,i}=\frac{p_{i}+M^{-1}q_{i}}{2}.$ $$

The seam’s reflection symmetry energy $E_{\mathrm{seam}}$ is then defined as:

$$ (8) $E_{\mathrm{seam}}=\omega_{\mathrm{seam}}\left(\sum_{p_{i}\in\mathcal{P}}(p_{i}-p_{t,i})^{2}+\sum_{q_{i}\in\mathcal{Q}}(q_{i}-q_{t,i})^{2}\right).$ $$

In our general framework, patches are cut and flattened progressively. Consequently, upon flattening a given patch, the energy $E_{\mathrm{seam}}$ may not be defined for all seams if the mirroring patch was not flattened previously. To address this, we jointly optimize $E_{\mathrm{seam}}$ for all patches during the final parameterization.

Figure: (a)
Refer to caption: /html/2202.10272/assets/img/semisphereviz2_shear.png

#### Dart symmetry.

Darts are similar to seams, but the two matching parts belong to the same pattern piece and meet at the tip, so the tip vertex is not duplicated. Ideally, this vertex lies on the dart’s symmetry axis in 2D. We use a similar energy to Eq. ([8](#S4.E8)) for darts but modify the target point definition.
We define the set of midpoints $\mathcal{R}$, consisting of $r_{i}=(p_{i}+q_{i})/2$, and find the target symmetry axis by computing the best fitting line to $\mathcal{R}$ while being constrained to pass through the tip vertex.
Then, we compute symmetric point sets $\overline{\mathcal{P}}$ and $\overline{\mathcal{Q}}$ and define an energy term $E_{\mathrm{dart}}$ equivalently to $E_{\mathrm{seam}}$, weighted by $\omega_{\mathrm{dart}}$ and with target positions

$$ (9) $\forall i\in[1\ldots n],\ \ q_{t,i}=\frac{\overline{p_{i}}+q_{i}}{2}\text{ , }p_{t,i}=\frac{p_{i}+\overline{q_{i}}}{2}.$ $$

Figure: Figure 15. Dart and seam reflection symmetry (Sec. [4.3](#S4.SS3)) is essential for producing workable patterns with reflective cuts.
Refer to caption: /html/2202.10272/assets/img/reflec/skirt_without.png

Figure: Figure 16. A skirt pattern piece (Fig. [2](#S2.F2), top) is split by a dart, resulting in conflicting desired grain alignment on each side. The global rotation chosen by our parameterization accommodates for both sides, despite being optimal for only a small area. On the left we show a histogram of the signed angle between the warp direction and the prescribed desired alignment.
Refer to caption: /html/2202.10272/assets/img/align_graph/align_viz3.png

#### Grain alignment.

Traditional garment patterns align the grain (warp) direction with the vertical axis in the final garment, or the “centerline” direction of a body part, such as along the sleeves. This ensures predictable and symmetric draping. In some cases, the tailor may perform a bias cut instead, favoring another alignment axis in specific regions in order to allow shearing on the main stress direction. In our framework, we allow the definition of a desired 3D alignment axis ${a^{\prime}}$ and rotate the 2D patches such that their warp aligns with ${a^{\prime}}$. This is first performed on a per-triangle basis by computing the projection
of ${a^{\prime}}$ on the triangle plane in 3D and transporting it to ${a}$ on the 2D triangle. This defines a per-triangle desired axis, and we compute the best-fit global axis $a_{\mathrm{opt}}$ that the $V$ direction should align to. Instead of including this in the energy formulation, we simply perform this alignment step in each iteration. Fig. [16](#S4.F16) shows an example of grain alignment.

#### Optimization.

We employ a local-global iteration strategy similar to (Sorkine and Alexa, 2007; Liu
et al., 2008) to compute a $UV$ mapping of a given pattern piece by minimizing the distortion measure

$$ (10) $E_{\mathrm{textile}}=E_{\mathrm{stretch,u}}+E_{\mathrm{stretch,v}}+E_{\mathrm{rigid}}+E_{\mathrm{seam}}+E_{\mathrm{dart}}$ $$

and accounting for grain alignment.
As the initial guess we use the least squares conformal map of (Lévy et al., 2002), orienting the 2D patch such that the $V$ axis matches the prescribed grain direction. We then compute the expressions in all the energy terms as described above (the local step), and solve for the updated parameterization of the pattern piece by minimizing $E_{\mathrm{textile}}$ w.r.t. the $UV$ coordinates of the mesh vertices. This global step amounts to efficiently solving a sparse linear system, because $E_{\mathrm{textile}}$ is quadratic in the $UV$’s. We iterate the local and global steps, reinstating the grain alignment in each local step. For all tested meshes, 5 iterations were sufficient to compute a satisfactory estimate and 20 iterations to converge.

The fabric properties are expressed by the weights $\omega$; the default values in our examples are $(\omega_{\mathrm{stretch}},\omega_{\mathrm{rigid}},\omega_{\mathrm{dart}},\omega_{\mathrm{seam}})=(5,1,5,5)$.
Fig. [14](#S4.F14) shows the effects of varying the distortion weights, while Fig. [15](#S4.F15) illustrates the seam and dart reflection symmetry constraints.

Figure: Figure 17. Our method works also for non-humanoid shapes, such as a dog (top) or a kangaroo (bottom).
Refer to caption: /html/2202.10272/assets/img/Animals/animals2.png

Figure: Figure 18. A sequence of interactive adjustments of to the patch layout. The designer adds sketches (in red), and our method computes seams that interpolate the sketched curves while generating a valid sewing pattern.
Refer to caption: /html/2202.10272/assets/img/interactive/interactivity_paraf.png

Our parameterization naturally generalizes to the case where multiple poses are considered, as only the right side of the system in the global step depends on the 3D mesh. At each iteration, we thus compute the right-hand side for each pose considered, and use the average in the solve.

Note that our parameterization method does not guarantee bijectivity. Self-intersections can thus occur during patch layout computation when cuts are introduced in non-satisfactory locations. Our patch decomposition method takes advantage of this by discarding cuts that lead to non-injective patches. This significantly speeds up our exploration of possible cuts, as only promising solutions are considered. The final pattern produced is guaranteed to contain no overlaps, as all the cuts passed our self-intersection test.

Figure: Figure 19. Automatic pattern layout generation for a shirt model, varying the two parameters that control the decomposition: the maximum number of corners per patch $C$ and the maximum stretch $s_{\mathrm{max}}$.
Refer to caption: /html/2202.10272/assets/img/Different_Param/8_005_3D.png

Figure: Figure 20. A tight-fitting dress pattern for a scanned body shape.
Refer to caption: /html/2202.10272/assets/img/NonStandard/non_standard_size_3D.png

## 5. Results

We integrate our framework into an interactive editor that allows the designer to control the result and explore the different sewing patterns generated by our system.
The main two main parameters exposed to the user are the maximum number of corners per pattern piece, $C$, and the maximum allowed stretch $s_{\mathrm{max}}$. Fig. [19](#S4.F19) shows the effect of these parameters on a simple example. The higher the number of permitted corners, the fewer patches appear in the patch decomposition. However, the patches are then more complex than the ones generated with fewer corners and potentially require more handling when sewing. Similarly, the maximum allowed distortion is also related to the number of patches inserted. Intuitively, more patches are needed to comply with a stricter distortion threshold, as seen in Fig. [19](#S4.F19). In Fig. [21](#S5.F21), we show how our system is able to match the maximal prescribed stretch, and we also measure the parameterization distortion for each pattern piece using the ARAP energy $E_{\mathrm{rigid}}$ (Liu
et al., 2008), showing that we achieve good results also in terms of isometry.

The designer can also adjust the patch layout using sketches on the 3D garment surface, see Fig. [18](#S4.F18) and the accompanying video. The initial cross-field is updated after each sketch to allow the patch decomposition to conform to the newly inserted constraints. We handle sketches in the same manner as Pietroni et al. (2021) handle sharp features. Our method allows real-time editing for small models (up to 3000 triangles) and takes a few seconds to process larger models.

Our method works on arbitrary input garments, both loose (Fig. [1](#S0.F1)) and tight-fitting (Fig. [20](#S4.F20)). Our framework can be used to fabricate personalized garments based on 3D scans (see Figures [23](#S5.F23), [24](#S5.F24)) or even garments for animals (see Fig. [17](#S4.F17)). When multiple target poses of the 3D garment are available, our method adapts the sewing pattern to accommodate the motion, see Fig. [22](#S5.F22).

We fabricated two examples: a wetsuit and a pair of leggings (see Figures [23](#S5.F23) and [24](#S5.F24)). Both manufactured garments exhibit an excellent fit, demonstrating the feasibility of our approach. The wetsuit is made of neoprene and assembled with an overlocker machine, whereas the leggings are made from 90% polyester and 10% spandex. The wetsuit pattern has an intricate structure and took around 5 hours for a single person to cut and sew, whereas the leggings only took about an hour.

Figure: Figure 21. We measure the fabric stress and the ARAP energy on one of our patterns, showing that we achieve good results in terms of both measures.
Refer to caption: /html/2202.10272/assets/img/distortion/Fig2_002.png

Figure: Figure 22. For animated models, we use the multi-scale principal curvature directions integrated though the frames to derive the cross-field. Similarly, we add per-frame energy terms in the patch parameterization. Having multiple target poses of the garment helps the method to place seams that adapt to highly deforming areas, such as the area where the leg connects to the torso (see the result on the right), which would otherwise not be noticed (see the result computed based on the rest shape alone on the left).
Refer to caption: /html/2202.10272/assets/img/Animation/NoAnim_deco0.png

Figure: Figure 23. A fabricated wetsuit. The input 3D garment model is based on a 3D scan of the subject. The open collar will be replaced with a zipper in the final garment. It took around 5 hours for a single person to prepare the pieces, cut, and sew.
Refer to caption: /html/2202.10272/assets/img/fabrication/fabric_illus.png

Figure: Figure 24. Tight-fitting trousers assembled and fabricated in around one hour.
Refer to caption: /html/2202.10272/assets/img/fabrication/leggings_viz.png

## 6. Conclusions

We present a pipeline that generates computational sewing patterns directly from 3D digital garments. The key novelty is our algorithm for shape segmentation and patch flattening that explicitly focuses on garment fabrication. By the usage of darts and taking into account the anisotropic properties of woven fabric, grain alignment and seam symmetry, our approach gives the designer the tools to produce (automatically and interactively) the 2D patterns to make the desired garment. We show that our algorithm produces patterns that are indeed suitable for fabrication of a custom, digitally designed garment. Our results also show that the pipeline can be applied to generate dewing patterns for tight-fitting and loose garments, for all types of body shapes, including animals.

#### Limitations and future work.

Although our approach is quite robust, it has some limitations. If the input mesh is the result of a simulation or a scan of a physical garment, the wrinkles can introduce noise in the guiding field.
Moreover, since the field tracing approach is greedy, the distortion in the final parameterization of the pattern pieces may differ from the one computed during the patch decomposition stage. This is because the seam reflection constraints cannot be incorporated before all patches are computed. In practice, we never encountered this problem. Also owing to the greedy nature of our method, a slight difference in the input garment or user constraints can result in a significantly different pattern. Finally, in future work we would like to include seam allowance constraints in our framework, to further secure easy and robust fabrication of the resulting sewing patterns.

## References

- (1)
- Alldieck
et al. (2021)
Thiemo Alldieck, Hongyi
Xu, and Cristian Sminchisescu.
2021.
imGHUM: Implicit Generative Models of 3D Human
Shape and Articulated Pose. In *Proc. ICCV*.
5461–5470.
[https://arxiv.org/abs/2108.10842](https://arxiv.org/abs/2108.10842)
- Bang
et al. (2021)
Seungbae Bang, Maria
Korosteleva, and Sung-Hee Lee.
2021.
Estimating Garment Patterns from Static Scan Data.
*Computer Graphics Forum*
40 (05 2021).
[https://doi.org/10.1111/cgf.14272](https://doi.org/10.1111/cgf.14272)
- Bartle et al. (2016)
Aric Bartle, Alla
Sheffer, Vladimir G. Kim, Danny M.
Kaufman, Nicholas Vining, and Floraine
Berthouzoz. 2016.
Physics-Driven Pattern Adjustment for Direct 3D
Garment Editing.
*ACM Trans. Graph.* 35,
4, Article 50 (jul
2016), 11 pages.
[https://doi.org/10.1145/2897824.2925896](https://doi.org/10.1145/2897824.2925896)
- Binninger* et al. (2021)
Alexandre Binninger*,
Floor Verhoeven*, Philipp Herholz, and
Olga Sorkine-Hornung. 2021.
Developable Approximation via Gauss Image
Thinning.
*Computer Graphics Forum (proceedings of SGP
2021)* 40, 5 (2021),
289–300.
[https://doi.org/10.1111/cgf.14374](https://doi.org/10.1111/cgf.14374)
- Bogo
et al. (2014)
Federica Bogo, Javier
Romero, Matthew Loper, and Michael J
Black. 2014.
FAUST: Dataset and evaluation for 3D mesh
registration. In *Proc. CVPR*.
3794–3801.
- Bommes et al. (2013)
David Bommes, Bruno
Lévy, Nico Pietroni, Enrico Puppo,
Cláudio T. Silva, Marco Tarini,
and Denis Zorin. 2013.
Quad-Mesh Generation and Processing: A Survey.
*Comput. Graph. Forum* 32,
6 (2013), 51–76.
- Bommes
et al. (2009)
David Bommes, Henrik
Zimmer, and Leif Kobbelt.
2009.
Mixed-integer quadrangulation.
*ACM Trans. Graph.* 28,
3 (2009), 77.
- Brouet
et al. (2012)
Remi Brouet, Alla
Sheffer, Laurence Boissieux, and
Marie-Paule Cani. 2012.
Design Preserving Garment Transfer.
*ACM Trans. Graph.* 31,
4, Article 36 (jul
2012), 11 pages.
[https://doi.org/10.1145/2185520.2185532](https://doi.org/10.1145/2185520.2185532)
- Campen (2017)
Marcel Campen.
2017.
Partitioning Surfaces Into Quadrilateral Patches:
A Survey.
*Comput. Graph. Forum* 36,
8 (2017), 567–588.
- Campen
et al. (2012)
Marcel Campen, David
Bommes, and Leif Kobbelt.
2012.
Dual loops meshing: quality quad layouts on
manifolds.
*ACM Trans. Graph.* 31,
4 (2012), 110:1–110:11.
- Chen (1998)
Jocelyn Hua-Chu Chen.
1998.
An investigation into 3-Dimensional Garment Pattern
Design.
*PhD Thesis, Nottingham Trent University, UK*
(1998).
- Chen
et al. (2015)
Xiaowu Chen, Bin Zhou,
Feixiang Lu, Lin Wang,
Lang Bi, and Ping Tan.
2015.
Garment Modeling with a Depth Camera.
*ACM Trans. Graph.* 34,
6, Article 203 (oct
2015), 12 pages.
[https://doi.org/10.1145/2816795.2818059](https://doi.org/10.1145/2816795.2818059)
- CLO (2022)
CLO. 2022.
clo3d.com.
[https://www.clo3d.com](https://www.clo3d.com).
- Cordier
et al. (2003)
Frederic Cordier, Hyewon
Seo, and Nadia Magnenat-Thalmann.
2003.
Made-to-measure technologies for an online clothing
store.
*IEEE Computer graphics and Applications*
23, 1 (2003),
38–48.
- Decaudin et al. (2006)
Philippe Decaudin, Dan
Julius, Jamie Wither, Laurence
Boissieux, Alla Sheffer, and
Marie-Paule Cani. 2006.
Virtual garments: A fully geometric approach for
clothing design.
*Comput. Graph. Forum*
25, 3 (2006),
625–634.
- Diamanti et al. (2014)
Olga Diamanti, Amir
Vaxman, Daniele Panozzo, and Olga
Sorkine-Hornung. 2014.
Designing *N*-PolyVector Fields with Complex
Polynomials.
*Comput. Graph. Forum* 33,
5 (2014), 1–11.
- Hormann
et al. (2007)
Kai Hormann, Bruno Lévy,
and Alla Sheffer. 2007.
Mesh Parameterization: Theory and Practice. In
*ACM SIGGRAPH Course Notes*.
- Huang
et al. (2012)
H.Q. Huang, P.Y. Mok,
Y.L. Kwok, and J.S. Au.
2012.
Block pattern generation: From parameterizing human
bodies to fit feature-aligned and flattenable 3D garments.
*Computers in Industry* 63,
7 (2012), 680–691.
[https://doi.org/10.1016/j.compind.2012.04.001](https://doi.org/10.1016/j.compind.2012.04.001)
- Ion et al. (2020)
Alexandra Ion, Michael
Rabinovich, Philipp Herholz, and Olga
Sorkine-Hornung. 2020.
Shape Approximation by Developable Wrapping.
*ACM Transactions on Graphics (proceedings of
SIGGRAPH ASIA)* 39, 6
(2020).
[https://doi.org/10.1145/3414685.3417835](https://doi.org/10.1145/3414685.3417835)
- Jiang
et al. (2017)
Zhongshi Jiang, Scott
Schaefer, and Daniele Panozzo.
2017.
Simplicial Complex Augmentation Framework for
Bijective Maps.
*ACM Trans. Graph.* 36,
6, Article 186 (nov
2017), 9 pages.
[https://doi.org/10.1145/3130800.3130895](https://doi.org/10.1145/3130800.3130895)
- Julius
et al. (2005)
Dan Julius, Vladislav
Kraevoy, and Alla Sheffer.
2005.
D-charts: Quasi-developable mesh segmentation.
*Computer Graphics Forum*
24, 3 (2005),
581–590.
- Korosteleva and
Lee (2021)
Maria Korosteleva and
Sung-Hee Lee. 2021.
Generating Datasets of 3D Garments with Sewing
Patterns. In *NeurIPS 2021 Datasets and Benchmarks
Track*.
- Kwok
et al. (2015)
Tsz Ho Kwok, Yan-Qiu
Zhang, Charlie Wang, Yong-Jin Liu, and
Kai Tang. 2015.
Styling Evolution for Tight-Fitting Garments.
*IEEE Transactions on Visualization and
Computer Graphics* 22 (01
2015).
[https://doi.org/10.1109/TVCG.2015.2446472](https://doi.org/10.1109/TVCG.2015.2446472)
- Lévy et al. (2002)
Bruno Lévy, Sylvain
Petitjean, Nicolas Ray, and Jérome
Maillot. 2002.
Least Squares Conformal Maps for Automatic Texture
Atlas Generation.
*ACM Trans. Graph.* 21,
3 (jul 2002),
362–371.
[https://doi.org/10.1145/566654.566590](https://doi.org/10.1145/566654.566590)
- Lévy et al. (2002)
Bruno Lévy, Sylvain
Petitjean, Nicolas Ray, and
Jérôme Maillot.
2002.
Least squares conformal maps for automatic texture
atlas generation.
*ACM Trans. Graph.* 21,
3 (2002), 362–371.
- Li
et al. (2018a)
Minchen Li, Danny M.
Kaufman, Vladimir G. Kim, Justin
Solomon, and Alla Sheffer.
2018a.
OptCuts: Joint Optimization of Surface Cuts and
Parameterization.
*ACM Transactions on Graphics*
37, 6 (2018).
[https://doi.org/10.1145/3272127.3275042](https://doi.org/10.1145/3272127.3275042)
- Li
et al. (2018b)
Minchen Li, Alla Sheffer,
Eitan Grinspun, and Nicholas Vining.
2018b.
FoldSketch: Enriching Garments with Physically
Reproducible Folds.
*ACM Transaction on Graphics*
37, 4 (2018).
[https://doi.org/10.1145/3197517.3201310](https://doi.org/10.1145/3197517.3201310)
- Liu et al. (2018)
Kaixuan Liu, Xianyi Zeng,
Pascal Bruniaux, Xuyuan Tao,
Xiaofeng Yao, Victoria Li, and
Jianping Wang. 2018.
3D interactive garment pattern-making technology.
*Computer-Aided Design*
104 (2018), 113–124.
[https://doi.org/10.1016/j.cad.2018.07.003](https://doi.org/10.1016/j.cad.2018.07.003)
- Liu
et al. (2008)
Ligang Liu, Lei Zhang,
Yin Xu, Craig Gotsman, and
Steven J. Gortler. 2008.
A Local/Global Approach to Mesh Parameterization.
In *Proceedings of the Symposium on Geometry
Processing* (Copenhagen, Denmark) *(SGP ’08)*.
Eurographics Association, Goslar,
DEU, 1495–1504.
- Livesu et al. (2020)
Marco Livesu, Nico
Pietroni, Enrico Puppo, Alla Sheffer,
and Paolo Cignoni. 2020.
LoopyCuts: practical feature-preserving block
decomposition for strongly hex-dominant meshing.
*ACM Trans. Graph.* 39,
4 (2020), 121.
- Loper et al. (2015)
Matthew Loper, Naureen
Mahmood, Javier Romero, Gerard
Pons-Moll, and Michael J. Black.
2015.
SMPL: A Skinned Multi-Person Linear Model.
*ACM Trans. Graph.* 34,
6 (Oct. 2015),
248:1–248:16.
- McCann et al. (2016)
James McCann, Lea
Albaugh, Vidya Narayanan, April Grow,
Wojciech Matusik, Jennifer Mankoff, and
Jessica Hodgins. 2016.
A Compiler for 3D Machine Knitting.
*ACM Trans. Graph.* 35,
4, Article 49 (jul
2016), 11 pages.
[https://doi.org/10.1145/2897824.2925940](https://doi.org/10.1145/2897824.2925940)
- McCartney
et al. (2005)
J. McCartney, B.K. Hinds,
and K.W. Chong. 2005.
Pattern flattening for orthotropic materials.
*Computer-Aided Design* 37,
6 (2005), 631–644.
[https://doi.org/10.1016/j.cad.2004.09.006](https://doi.org/10.1016/j.cad.2004.09.006)
CAD Methods in Garment Design.
- McCartney
et al. (2000)
J. McCartney, B.K. Hinds,
B.L. Seow, and D. Gong.
2000.
An energy based model for the flattening of woven
fabrics.
*Journal of Materials Processing Tech.*
107, 1-3 (2000),
312–318.
- Meng
et al. (2012)
Yuwei Meng, Charlie CL
Wang, and Xiaogang Jin.
2012.
Flexible shape control for automatic resizing of
apparel products.
*Computer-Aided Design* 44,
1 (2012), 68–76.
- Montes
et al. (2020)
Juan Montes, Bernhard
Thomaszewski, Sudhir Mudur, and Tiberiu
Popa. 2020.
Computational Design of Skintight Clothing.
*ACM Trans. Graph.* 39,
4, Article 105 (jul
2020), 12 pages.
[https://doi.org/10.1145/3386569.3392477](https://doi.org/10.1145/3386569.3392477)
- Narayanan et al. (2018)
Vidya Narayanan, Lea
Albaugh, Jessica Hodgins, Stelian Coros,
and James McCann. 2018.
Automatic Machine Knitting of 3D Meshes.
*ACM Trans. Graph.* 37,
3, Article 35 (Aug.
2018), 15 pages.
[https://doi.org/10.1145/3186265](https://doi.org/10.1145/3186265)
- Narayanan* et al. (2019)
Vidya Narayanan*, Kui
Wu*, Cem Yuksel, and Jim McCann.
2019.
Visual Knitting Machine Programming.
*ACM Transactions on Graphics (Proceedings of
SIGGRAPH 2019)* 38, 4, Article
63 (jul 2019),
13 pages.
[https://doi.org/10.1145/3306346.3322995](https://doi.org/10.1145/3306346.3322995)
(*Joint First Authors).
- Nayak and Padhye (2017)
Rajkishore Nayak and
Rajiv Padhye. 2017.
*Automation in Garment Manufacturing*.
Woodhead Publishing.
- Nuvoli et al. (2019)
Stefano Nuvoli, Alex
Hernandez, Claudio Esperança,
Riccardo Scateni, Paolo Cignoni, and
Nico Pietroni. 2019.
QuadMixer: layout preserving blending of
quadrilateral meshes.
*ACM Trans. Graph* 38,
6 (2019), 180:1–180:13.
- Optitex (2022)
Optitex. 2022.
optitex.com.
[https://optitex.com](https://optitex.com).
- Osman
et al. (2020)
Ahmed A A Osman, Timo
Bolkart, and Michael J. Black.
2020.
STAR: A Sparse Trained Articulated Human Body
Regressor. In *European Conference on Computer
Vision (ECCV)*. 598–613.
[https://star.is.tue.mpg.de](https://star.is.tue.mpg.de)
- Panozzo
et al. (2012)
Daniele Panozzo, Yaron
Lipman, Enrico Puppo, and Denis
Zorin. 2012.
Fields on symmetric surfaces.
*ACM Trans. Graph* 31,
4 (2012), 111:1–111:12.
- Panozzo
et al. (2010)
D. Panozzo, Enrico Puppo,
and L. Rocca. 2010.
Efficient multi-scale curvature and crease
estimation.
*2nd International Workshop on Computer
Graphics, Computer Vision and Mathematics, GraVisMa 2010 - Workshop
Proceedings* (01 2010),
9–16.
- Pietroni et al. (2021)
Nico Pietroni, Stefano
Nuvoli, Thomas Alderighi, Paolo Cignoni,
and Marco Tarini. 2021.
Reliable Feature-Line Driven Quad-Remeshing.
*ACM Trans. Graph.* 40,
4, Article 155 (jul
2021), 17 pages.
[https://doi.org/10.1145/3450626.3459941](https://doi.org/10.1145/3450626.3459941)
- Pietroni et al. (2016)
Nico Pietroni, Enrico
Puppo, Giorgio Marcias, Roberto
Scopigno, and Paolo Cignoni.
2016.
Tracing Field-Coherent Quad Layouts.
*Comput. Graph. Forum* 35,
7 (2016), 485–496.
- Pons-Moll
et al. (2017)
Gerard Pons-Moll, Sergi
Pujades, Sonny Hu, and Michael J
Black. 2017.
ClothCap: Seamless 4D clothing capture and
retargeting.
*ACM Trans. Graph.* 36,
4 (2017), 1–15.
- Poranne et al. (2017)
Roi Poranne, Marco
Tarini, Sandro Huber, Daniele Panozzo,
and Olga Sorkine-Hornung.
2017.
Autocuts: Simultaneous Distortion and Cut
Optimization for UV Mapping.
*ACM Transactions on Graphics (proceedings of
ACM SIGGRAPH ASIA)* 36, 6
(2017).
- Rabinovich et al. (2017)
Michael Rabinovich, Roi
Poranne, Daniele Panozzo, and Olga
Sorkine-Hornung. 2017.
Scalable Locally Injective Mappings.
*ACM Transactions on Graphics*
36, 2 (April
2017), 16:1–16:16.
- Razafindrazaka et al. (2015)
Faniry H. Razafindrazaka,
Ulrich Reitebuch, and Konrad Polthier.
2015.
Perfect Matching Quad Layouts for Manifold Meshes.
*Comput. Graph. Forum* 34,
5 (2015), 219–228.
- Robson
et al. (2011)
C. Robson, R. Maharik,
A. Sheffer, and N. Carr.
2011.
Context-Aware Garment Modeling from Sketches.
*Computers & Graphics (Proc. SMI 2011)*
35, 3 (2011),
604–613.
- Rose
et al. (2007)
Kenneth Rose, Alla
Sheffer, Jamie Wither, Marie-Paule Cani,
and Boris Thibert. 2007.
Developable surfaces from arbitrary sketched
boundaries. In *Proc. Symposium on Geometry
Processing*. Eurographics Association, 163–172.
- Sawhney and Crane (2017)
Rohan Sawhney and Keenan
Crane. 2017.
Boundary First Flattening.
*CoRR* abs/1704.06873
(2017).
arXiv:1704.06873
[http://arxiv.org/abs/1704.06873](http://arxiv.org/abs/1704.06873)
- Sharp and Crane (2018)
Nicholas Sharp and
Keenan Crane. 2018.
Variational Surface Cutting.
*j-TOG* 37,
4 (2018).
- Sheffer et al. (2004)
Alla Sheffer, Bruno
Lévy, Maxim Mogilnitsky, and
Alexander Bogomyakov. 2004.
ABF++ : Fast and Robust Angle Based Flattening.
*ACM Trans. Graph.* 24,
2 (2004), 311–330.
[https://hal.inria.fr/inria-00105689](https://hal.inria.fr/inria-00105689)
- SizeGermany (2020)
SizeGermany.
2020.
SizeGermany.
[https://portal.sizegermany.de](https://portal.sizegermany.de).
- Sorkine and Alexa (2007)
Olga Sorkine and Marc
Alexa. 2007.
As-Rigid-As-Possible Surface Modeling. In
*Geometry Processing*,
Alexander Belyaev and
Michael Garland (Eds.). The
Eurographics Association.
[https://doi.org/10.2312/SGP/SGP07/109-116](https://doi.org/10.2312/SGP/SGP07/109-116)
- Sorkine et al. (2002)
Olga Sorkine, Daniel
Cohen-Or, Rony Goldenthal, and Dani
Lischinski. 2002.
Bounded-distortion piecewise mesh
parameterization. In *Proceedings of IEEE
Visualization* (Boston, Massachusetts). IEEE Computer
Society, 355–362.
- Sorkine-Hornung and Rabinovich (2016)
Olga Sorkine-Hornung and
Michael Rabinovich. 2016.
Least-Squares Rigid Motion Using SVD.
Technical note.
- Stein
et al. (2018)
Oded Stein, Eitan
Grinspun, and Keenan Crane.
2018.
Developability of triangle meshes.
*ACM Trans. Graph.* 37,
4 (2018).
- TiltBrush (2022)
TiltBrush.
2022.
www.tiltbrush.com.
[https://www.tiltbrush.com/](https://www.tiltbrush.com/).
- Turquin et al. (2007)
Emmanuel Turquin, Jamie
Wither, Laurence Boissieux, Marie-Paule
Cani, and John F Hughes.
2007.
A sketch-based interface for clothing virtual
characters.
*IEEE Computer Graphics and Applications*
27, 1 (2007).
- Umetani et al. (2011)
Nobuyuki Umetani, Danny M.
Kaufman, Takeo Igarashi, and Eitan
Grinspun. 2011.
Sensitive Couture for Interactive Garment Editing
and Modeling.
*ACM Transactions on Graphics (SIGGRAPH
2011)* 30, 4 (2011).
- Vaxman et al. (2017)
Amir Vaxman, Marcel
Campen, Olga Diamanti, David Bommes,
Klaus Hildebrandt, Mirela Ben-Chen,
and Daniele Panozzo. 2017.
Directional field synthesis, design, and
processing. In *SIGGRAPH ’17 Courses*.
12:1–12:30.
- Vidaurre et al. (2020)
Raquel Vidaurre, Igor
Santesteban, Elena Garces 0001, and Dan
Casas. 2020.
Fully Convolutional Graph Neural Networks for
Parametric Virtual Try-On.
*CoRR* abs/2009.04592
(2020).
- Wang
et al. (2005b)
Charlie C.L. Wang, Yu
Wang, and Matthew M.F. Yuen.
2005b.
Design automation for customized apparel products.
*Computer-Aided Design* 37,
7 (1 June 2005),
675–691.
[https://doi.org/10.1016/j.cad.2004.08.007](https://doi.org/10.1016/j.cad.2004.08.007)
- Wang
et al. (2005a)
Charlie CL Wang, Kai
Tang, and Benjamin ML Yeung.
2005a.
Freeform surface flattening based on fitting a
woven mesh model.
*Computer-Aided Design* 37,
8 (2005), 799–814.
- Wang (2018)
Huamin Wang.
2018.
Rule-Free Sewing Pattern Adjustment with Precision
and Efficiency.
*ACM Trans. Graph.* 37,
4, Article 53 (jul
2018), 13 pages.
[https://doi.org/10.1145/3197517.3201320](https://doi.org/10.1145/3197517.3201320)
- Wang
et al. (2018)
Tuanfeng Y. Wang, Duygu
Ceylan, Jovan Popovic, and Niloy J.
Mitra. 2018.
Learning a Shared Shape Space for Multimodal
Garment Design.
*ACM Trans. Graph.* 37,
6 (2018), 1:1–1:14.
[https://doi.org/10.1145/3272127.3275074](https://doi.org/10.1145/3272127.3275074)
- Wibowo
et al. (2012)
Amy Wibowo, Daisuke
Sakamoto, Jun Mitani, and Takeo
Igarashi. 2012.
DressUp: a 3D interface for clothing design with a
physical mannequin. In *Proc. International
Conference on Tangible, Embedded and Embodied Interaction*. ACM,
99–102.
- Wolff
et al. (2019)
Katja Wolff, Philipp
Herholz, and Olga Sorkine-Hornung.
2019.
Reﬂection Symmetry in Textured Sewing Patterns.
In *Proceedings of the Symposium on Vision, Modeling
and Visualization (VMV)*. Eurographics Association.
- Wolff et al. (2021)
Katja Wolff, Philipp
Herholz, Verena Ziegler, Frauke Link,
Nico Brügel, and Olga
Sorkine-Hornung. 2021.
3D Custom Fit Garment Design with Body Movement.
*CoRR* abs/2102.05462
(2021).
arXiv:2102.05462
[https://arxiv.org/abs/2102.05462](https://arxiv.org/abs/2102.05462)
- Wu
et al. (2019)
Kui Wu, Hannah Swan,
and Cem Yuksel. 2019.
Knittable Stitch Meshes.
*ACM Transactions on Graphics*
38, 1, Article 10
(jan 2019), 13 pages.
[https://doi.org/10.1145/3292481](https://doi.org/10.1145/3292481)
- Yuksel
et al. (2012)
Cem Yuksel, Jonathan M.
Kaldor, Doug L. James, and Steve
Marschner. 2012.
Stitch Meshes for Modeling Knitted Clothing with
Yarn-Level Detail.
*ACM Transactions on Graphics (Proceedings of
SIGGRAPH 2012)* 31, 3, Article
37 (2012), 12 pages.
[https://doi.org/10.1145/2185520.2185533](https://doi.org/10.1145/2185520.2185533)