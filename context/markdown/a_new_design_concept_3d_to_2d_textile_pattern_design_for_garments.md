<!-- markdownlint-disable -->
# A New Design Concept: 3D to 2D Textile Pattern Design for Garments

Shufang Lu $^{1,2}$ , P.Y. Mok $^{2,*}$ , Xiaogang Jin $^{3}$

<sup>1</sup> College of Computer Science and Technology, Zhejiang University of Technology, China   
$^{2}$ Institute of Textiles and Clothing, The Hong Kong Polytechnic University, Hong Kong

$^{3}$ State Key Lab of CAD&CG, Zhejiang University, Hangzhou, China

*Corresponding author tel.: 852 2766 4442; fax: 852 2773 1432;

e-mail: tracy.mok@polyu.edu.hk

# Abstract

'The word "fashion" is synonymous with the word "change". Fashion begins with fabrics and fabrics begin with colour.' This famous remark/definition of 'fashion' must now be revised in the era of digital technology. In this paper, we propose a novel print design concept, from 3D garments to 2D textiles. By taking advantage of the cutting-edge developments in surface parameterisation, cloth simulation and texture assignment, we develop a computer system that allows designers to create figure-flattering prints directly onto 3D garments, and will output 2D pattern pieces with matched texture that are ready for digital printing and garment production. It reverses the traditional design process from 2D fabrics to 3D garments. The results produced by the proposed method guarantee textural continuity in both garment and pattern pieces. It not only releases apparel makers from the tedious work of matching texture along seams, but also provides users with a new tool to create one-of-a-kind fashion products by designing personalised prints.

Keywords: Apparel prints, textile pattern design, digital 3D painting, texture projection

# 1. Introduction

It is the very nature of every human being, from the ancient time, to beautify the body with attractive decorations, including clothing. It is the mission of every designer to satisfy such human desire that consequently keeps the momentum of growth in the fashion business. Fashion designers determine the silhouettes, colours, fabrics, trimmings, and details in the design process for each product [1]. The best figure-flattering design should have the ideal prints and colours to match silhouettes; nevertheless, fashion designers do not have full control of all these five elements in the design process. This is because the fashion business has one of the longest supply chains, taking over two years to move from raw materials of fibres, to yarns, to fabrics, and to apparel; each stage involves hundreds of operations.

![](images/4e8ea76cee09c886b6e4c8b4f1a041dec811919adcbb4330d9c6c656ff0a5a2b.jpg)  
Figure 1. Fashion and textile supply chain.

Textile and fashion design are two separate processes in the fashion supply chain. As shown in Figure 1, textile products typically pass through a complex lifecycle beginning with the production of the raw materials (fibres), to spinning of yarns, fabrication of cloth from yarns (by weaving or knitting), followed by a variety of processes to make the finished cloth more appealing, colourful and useful. Further stages in the supply chain include apparel product development and manufacture, and the final step of sale and logistics (transferring clothes to consumers). It is important to note that fabric manufacturing is done in bulk at least six months ahead of the design and production of clothing (fashion), and fabric design is done by another team of experts, textile designers, not fashion designers. As shown in Figure 1, fashion designers are not responsible for fabric design; rather, they choose the prints and colours of the fabrics to best present their products.

On the other hand, for a figure-flattering design, the prints on the clothing must follow the 3D body shape. Designers may wish to visualise the prints/pattern on the garment in 3D so as to have the best print arrangement. Although the final wearing effect is assessed in 3D, the upstream garment manufacturing and fabric print design are mainly completed on two-dimensional basis. It is the ideal case if the 3D wearing effects can be visualised before confirmation of which textile prints to use. However, this requires reversing the design process of textiles and clothing.

The current technology tries to tackle this problem with a technique called 'engineered' patterns, i.e. the careful placement of clothing pattern pieces on fabrics in the process of marker planning; when these pieces are cut and assembled into a garment, a degree of continuity is maintained in that the design flows unbroken around the body. Matching prints along seams is capable of helping disguise the seams of a garment [2]. Clothes with engineered patterns are perceived to be more luxurious and elegant [3] because matching prints demand a lot in terms of operators' skills and patience. Notable designers using such techniques include Mary Katrantzou and Alexander McQueen [4-5]. Other designers such as Sonia Delaunay, Emilio Pucci, Gianni Versace, and Basso & Brooke all favour engineered prints [3]. It is also an observed trend in recent catwalk shows, as illustrated in Figure 2, that textile prints are matched specifically to fit the form of a garment following the body contours of the wearer.

![](images/9cce822b26867e40ae0340c77e16da324e7a700d8296ca80cde590d13e9dadf3.jpg)  
Mary Katrantzou Spring 2013

![](images/a94f4363dc83d0da17ea9f7c3dbb8cd1cbc2590268f68fb603d2970710cd1935.jpg)  
Just Cavalli Fall 2013

![](images/17bda5fd9034c13d6b91b71bc505186412d972243b649879edbd6485d8f27fd6.jpg)  
Dolce & Gabbana Spring 2014

![](images/f945452878201638ffc86459eee53c95ea41e0bcfe28a14d26616a510a01f1ff.jpg)  
Valentino Spring 2015

![](images/0376ef5540c61dce7e3f49b2fef74622f1f1f7859b2d149aabac945c91589f24.jpg)  
Dolce & Gabbana Spring 2016

![](images/bf1bae529144782a2046babc2e90465980cfeae4611ad5eb7e23b7ba71ffed6d.jpg)  
Alexander McQueen Fall 2016

![](images/49e1f06489dc1a32de0e8043508009c94d19e1aea49196ddb3047e73374a283c.jpg)  
Schiaparelli Fall 2016

![](images/6ef01bf63ccb73a5cb29b46bb41bf737ea54abeb44a43e17b653927a41a3055f.jpg)  
Dolce & Gabbana Spring 2017   
Figure 2. Recent catwalk examples with matched prints [6].

Although print placement or engineered prints can obtain continuous patterns, they have a number of drawbacks. This will be further explained in Section 2.2. In this paper, a novel method

is proposed that allows the textile and fashion design processes to be reversed; a computer system is developed by which fashion designers can customise designs for individual customers and create figure-flattering prints on 3D garments. Taking advantage of cutting-edge digital printing technology, the 3D design can be digitally printed out on plain fabrics as pattern pieces with matched prints. The pattern pieces fulfil the apparel manufacturing requirements with proper seam allowance and correct grain. This method ensures continuity of prints along all seams on complex pattern pieces, and it also improves the fabric utilisation with correct grain alignment of all cut pieces on fabrics. The main contributions of the paper are summarised as follows.

A new design concept: traditionally, 3D garments are made of fabrics, and garments designed must fit the constraints of a patterned fabric. The method proposed in this paper is the reverse of the traditional garment-making process, where prints/patterns are designed on 3D garments directly, and the resulting pattern pieces with texture are printed on plain fabrics.

Intuitive, customised and figure-flattering design: designing textile patterns on a 3D garment directly is an intuitive way to show the final wearing effects. Designers can experiment and see the end-results without actually producing the garments. Designers can create one-of-a-kind fashion products with figure-flattering prints for individual customers.

Unbroken continuity of design eliminating tedious pattern placement: the textile patterns on the garment work with the shape and form of the garment. The continuity design flows unbrokenly around the body and is capable of helping disguise the seams of the apparel.

# 2. Literature review

Before discussion of our method, we give an overview of the manufacturing constraints in the textile and fashion industry, including the garment-making process and the textile printing. The recent developments in computer-aided clothing design are also reviewed in this section.

# 2.1 Sequential processes of garment manufacturing

Garment manufacturing is traditionally a sequential process that has a series of operations, including fashion design, pattern design, grading, marker making, spreading, cutting, sewing and assembly (see Figure 3).

![](images/d21ec36c413c7d9b8778da4b55d535cd895a7004b9e4fa8809d8a888067f0bdd.jpg)  
Figure 3. Fashion product development and garment manufacturing processes.

The first step in garment manufacturing is fashion design, which outputs design sketches with indications of materials, colours, and size details. After a garment is designed, the next step is pattern design, which creates pattern pieces. Traditionally, fashion design and pattern design are an iterative process called product development, involving a series of design adjustment, pattern drafting/alteration, and sample construction. The outputs from the product development process are detailed product specification including confirmed design sketches (Figure 4(a)) and production patterns (Figure 4(b)), in which seam allowance and grain lines are provided for later production operations of cutting and sewing. As shown in Figure 4(c), seam allowance is the narrow width of fabric between the seam line and the cut edge of the fabric. The grain line indicates either the lengthwise or crosswise grain in the fabric, i.e. the orientation of the yarns that make up the fabric.

After product development, grading is used to obtain pattern pieces of different sizes. Next, the pattern pieces are arranged for cutting in marker planning, and proper grain alignment in this operation is vitally important to ensure the finished garment assembled from the cut pieces drapes correctly and has the desired aesthetic and functional quality. As shown in Figure 4(d), the output from this operation is markers which provide the layouts of pattern pieces and are used as a guide for cutting. The next step of the process is spreading and cutting; in mass production, a large number of fabric plies are laid (spread) and cut before sent to downstream sewing lines. The last step is the sewing operation, where cut pieces are assembled to create the structure and detail of a garment.

![](images/64a8913f917c6037f871b9f55282188dd77572fdb4d15296fa2f678592076ed2.jpg)

![](images/a7c6dec5ee61ef00e2c592e15fa44b4776c201fca3952ecca3f7a9e1d0cb13ed.jpg)

![](images/33b3a69d1599c6e71c3e216d5fc40490768095f785b1f9ff10db50e8fb01bf76.jpg)

![](images/337062c6b0f75219c88445884fbb871c2642bdc7e7a50ab0ee1de63ba117334c.jpg)  
Figure 4.(a) Fashion sketches; (b) clothing pattern pieces; (c) zoomed details of patterns; (d) a marker example.

# 2.2 Printed fabrics: from traditional to digital

In traditional garment manufacturing, fabrics are cut one a plane and then assembled to form a three-dimensional garment. Fashion begins with fabrics and fabrics begin with colour. Printing refers to the method of colouring some areas of fabrics differently from others by using dyes, pigments and paints. The main methods of printing are batik, tie dye, band painting with mordants, block printing, screen printing, roller printing, transfer printing, and direct printing [8]. The design and production of textile printing are performed under considerable constraints: the industry is itself capital-intensive, and the use of dyes, pigments and paints and other fixation agents in the production lead to high manufacturing cost for every single new design. It implies large order size and long production runs are mandatory for printing. It also explains the long production cycle in the textile and clothing industry, as discussed in Section 1 and shown in

# Figure 1.

The introduction of digital textile printing in the early 1990s revolutionised the entire textile printing industry, from methods of creating and presenting designs to the ways in which they were realised. With the rapid development in new printers and inks, complex designs can now be printed directly onto textile media at very high speed. The restrictions that textile designers

have traditionally faced are now removed: they can work with thousands of colours and create designs with a high level of detail; the sampling timespan is drastically reduced from months to days, making one-off production as well as small print runs possible.

Table 1. Feasible and infeasible regions of engineered patterns [24, p.318].   

<table><tr><td>Key seams/location where pattern matching is expected in high-quality garment</td><td>Places where matching cannot be expected at any quality level</td></tr><tr><td>• Centre front and centre back
• Side seams
• Collar at centre back (for collars that open in front); collar at centre front (for collars that open at the back)
• Armscye at bust/chest level
• Shaped facing
• Sleeve plackets
• Pockets/pocket flags
• Two-piece outfits where they overlap, so that the patterns appear continuous from top to bottom
• Shoulder seams</td><td>• Seams with darts
• Any area that contains ease or gathers
• Yokes (if the yoke is a dart-substitute yoke)
• Armscye backs and fronts (above bust/chest level)
• Pattern pieces with varying angles
• Raglan sleeve
• Cuffs
• Waist seams and waistbands</td></tr></table>

Digital textile printing makes it more economically feasible to create prints specifically to fit within a garment. However, placement of digitally printed fabrics still faces the same problems as the printed fabrics created by traditional methods. A few of the drawbacks of engineered prints are as follows. First, since the pattern on fabrics is printed before the garment is made, the print stays the same regardless of the size of the garment. It means that the prints/designs may appear too large on smaller sized apparel, or appear too small on plus sized apparel. It thus may not achieve the best effect on customised design. Second, matching prints along seams lowers the fabric utilisation, and in turn increases the material costs. Third, the clothing pattern pieces often have irregular geometrical shapes. Placement of prints on a plane may result in matching prints along one seam, but not the other seams on the same piece. It is extremely tedious and time-consuming to match prints on a 2D plane, especially for styles with a complex design and a large number of pattern pieces (see Table 1). Fourth, the layout of pattern pieces on fabrics must take

into consideration of the desired grain alignment for the best drape effect of the finished garment [24], this makes matching prints on a plane even more challenging. It is important to note that the method proposed in our paper overcomes such problems by enabling print design on 3D garments and lowers the production cost by maximising fabric utilisation with advanced computer-aided design technologies.

# 2.3 Cutting-edge computer-aided 3D clothing design

The research of fashion related computer-aided design has blossomed in the past two decades because of the introduction of 3D computer-aided design (CAD) techniques in patternmaking. Traditionally, the function of computers is to act as a drawing tool, just like pen and ruler in paper-based patternmaking, to obtain clothing patterns in digital form. In recent years, some researchers worked on parametric patternmaking [25-28] and make-to-measure (MTM) patterns [29].

On the other hand, since the end of the twentieth century a large amount of work has been done on 3D clothing simulation [18-20], including customising 3D models [7], virtual assembling 2D digital patterns into a 3D virtual garment [30-31; 9, 21], and direct 3D garment editing [22]. For example, [9] placed pattern pieces interactively around a virtual human model along the process of virtual sewing. Compared with previous methods like the seminal work of [32], this method can realise complex designs. [12] proposed a novel cross parameterisation technique for mapping the garment pattern pieces on the human model surface, enabling easy style editing. [22] introduced a framework for direct editing garments in 3D space and producing simulation and manufacturing ready 2D patterns. [21] presented a system for automatically parsing sewing patterns and converting them into detailed 3D garment models for virtual characters. The aim of 3D clothing simulation is to visualise the wearing effect of apparel product on a 3D model before actual production of real sample [33].

Looking at the recent development in fashion CAD, it is not difficult to tell that most research efforts are focused on garment development, namely the pattern design, sampling and fitting [37-38], as shown in the yellow highlighted process in Figure 3. It means these new initiatives in fashion CAD again follow the traditional textile and fashion design processes (see Figure 1). With the advancement in digital textile printing and 3D computer-aided fashion design technologies, we propose in this paper to design print/pattern on 3D garments directly. To the best of our knowledge, there is no reported method for designing textile patterns directly on the 3D garment model on computers and generating accurate 2D printed patterns for production.

# 3. Methodology

# 3.1 System overview

We develop in this paper a computer system reversing the textile and fashion design process that fashion designers create figure-flattering prints directly on 3D garments and then obtain 2D pattern pieces with matched prints for later garment production. The system overview is shown in Figure 5, where key operations are highlighted in blue. First, a surface parameterisation and hybrid pop-up method is used to generate a 3D geometric garment model from 2D pattern pieces. Next, pattern pieces are arranged in 2D and the points on pieces are normalised as the texture coordinates for the corresponding vertices on the garment model. Then, two techniques including 3D digital painting and texture projection are used to design prints on the garment models. The wearing effects can be visualised by carrying out drape simulation. Finally, the textured pattern pieces are printed on fabric digitally with seam allowance added, they are then cut and sewn together as the final garment. The method is discussed in detail in the following subsections.

![](images/53fd72b650ccda9fb1c59e67038b5a51607f7f7a9e485e8ce03ca26db9064af1.jpg)  
Figure 5. From 3D garments to 2D textured patterns - a system overview.

# 3.2 Surface parameterisation and 3D garment generation

This method uses a 3D garment model as an input for print design, and aims to produce pattern pieces with texture for a real garment. There are different ways to obtain 3D garments. Some researchers have proposed designing garments directly in 3D space and later flattening the 3D garments to obtain 2D patterns [10-11]. However, because of the unavoidable distortion in the flattening process, the flattened pattern pieces are not good for garment manufacture. As reviewed in Section 2.3, most researchers virtually simulate 2D pattern pieces around a human

model to form 3D garments using the finite element method (FEM). However, the output 3D garments have folds and gathers formed in the physical-based drape simulation, making them unsuitable for designing prints on top.

Since the print design on the garment models will map onto the pattern pieces, low-distortion surface parameterisation between garment and patterns should be employed. Good parameterisation methods should fulfil the following requirements: (1) one-to-one correspondence should be established between the feature point of 2D pattern pieces and that of 3D garments; (2) the boundary curves in the 2D pattern pieces should have a similar length to the corresponding seams in the 3D garment; and (3) the garment model should retain the size of the 2D pattern pieces.

To generate a 3D garment ready for print design, we employ the method proposed by [12], which takes 2D clothing patterns for everyday production as input and forms a 3D geometrical garment model as output. It precisely defines one-to-one mapping between feature points on patterns and on the garment, and the boundary length and pattern size on the garment are restored to those of the original 2D pattern pieces. As a result, it ensures very low distortion between garment and pattern pieces. The process is described as follows.

Step One: initial form of garment generation by surface parameterisation. The process starts by defining the corresponding feature points on both the human mesh model and clothing pattern pieces. Ancillary patterns are then obtained from triangulating the 2D pattern pieces by the constrained Delaunay method (Figure 7(a)). Next, the human mesh model is segmented into several patches corresponding to these ancillary patterns. Finally, a cross-parameterisation procedure is carried out by a duplex mapping scheme. As a result, the initial form of the garment (Figure 7(b)) and a mapping relationship are generated.

Step Two: restoration to desired shape and size of the garment. An iterative hybrid pop-up method is employed to deform the initial garment. First, the initial form of the garment's silhouette (i.e. girth measurement at the hem level) is adjusted to the size of corresponding pattern pieces. Then, a physical simulation is performed along the normal direction to update the garment length. As a result, the position of the boundary vertices is changed. Finally, other vertices are updated by a geometrical reconstruction with the use of pyramid coordinates (see

Figure 6).

![](images/31a30fcf1b362d9faf30d5d3e7d78c19bcbad089b6875b67b2430e915571dc52.jpg)  
Figure 6. Pyramid coordinates of vertex $\nu^{3d}$ , where $\nu_{i}^{3d}$ is one of its neighbouring vertices.

The pyramid coordinate of vertex $\nu^{3d}$ is calculated as weight parameter $\omega_{i}^{3d}$ by:

$$
\omega_ {i} ^ {3 d} = \omega_ {i} ^ {2 d} (1 - \cos \theta_ {i}) \tag {1}
$$

where $\theta_{i}$ is the normal angle; $\omega_{i}^{2d}$ is the mean value coordinate weight, as defined by its neighbouring angle $\alpha_{i}$ and length $l_{i}$ as follows:

$$
\omega_ {i} ^ {2 d} = \frac {\tan \left(\alpha_ {i} / 2\right) + \tan \left(\alpha_ {i + 1} / 2\right)}{l _ {i}} \tag {2}
$$

where $l_{i} = |v^{2d} - v_{i}^{2d}|$

$$
\theta_ {i} = \left(\arcsin \frac {l}{| v ^ {3 d} - v _ {i} ^ {3 d} |} + \arcsin \frac {l}{| v ^ {2 d} - v _ {i} ^ {2 d} |}\right) / 2 \tag {3}
$$

where $|v^{3d} - v_i^{3d}|$ is the length between $v^{3d}$ and its neighbour $v_i^{3d}$ on the 3D shape of the current reconstruction iteration, $|v^{2d} - v_i^{2d}|$ is the corresponding length on the 2D pattern, $l$ is the length parameter value for the local coordinate of the current reconstruction iteration.

A sample blouse model consisting of nine patches each with a unique colour is shown in Figure 7(b). Each 2D pattern has the same colour as its corresponding patch on the 3D garment. Figure 7(a) shows the corresponding 2D pattern pieces. Figure 7(c) and (d) show the front view and the back view of the garment, respectively. It is important to note that the garment model is restored to the original size defined by the 2D clothing pattern pieces in this method. The area preservation guarantees that parameterisation between the desired shape of the garment and the clothing patterns brings little distortion. The normalised points on the 2D pattern pieces can be assigned with the texture coordinates of the corresponding vertices on the 3D garment.

Consequently, the texture designed on the 3D garment will automatically map to the 2D pattern pieces by means of this spatial relationship. It provides the basis for 3D painting and cylindrical texture projection described in Section 3.5.

![](images/6f8d4a68a54a409763954b44e28e2f23a8c79abe06d2b650f43a41037b8b867c.jpg)

![](images/25637b56ff371abfea6dd48dfc4627ed18e2023be19e78bc9fa8f8541bc69d26.jpg)

![](images/c515ce6b84c5c948068c0038d9e344cae564ecc0b111be2cf11cc966fa77e007.jpg)

![](images/c24bf1910d9e4b792ecf195cb8d83faeb9c5098c17e6936ec17284413d4be58a.jpg)  
Figure 7. (a) 2D clothing pattern pieces, (b) initial form of garment by pattern-to-model parameterisation (front view), and (c) (d) the desired shape of garment after hybrid pop-up (front and back view).

# 3.3 Pattern layout for a texture image: marker planning

Given the set of the 2D mesh pattern pieces obtained from the triangulation process described in Section 3.2, we next define a texture atlas by merging all pattern pieces on a 2D plane. The texture atlas enables the subsequent step of texture coordinate assignment for establishing one-to-one correspondence between the object coordinate and the texture coordinate. As discussed in Section 2.1, the process of finding a non-overlapping placement of the pattern pieces in a minimum enclosing rectangular area is called marker planning in garment manufacturing. The main objective of marker planning is to minimise the size of the marker and maximise fabric utilisation. In our method, the pattern pieces together with the matched texture are printed out by digital printers for garment production; minimising image (marker) size can also lower the texture memory consumption.

Finding the optimal layout of the patterns is known as the packing problem, which is a special instance of an NP-complete problem. This problem is studied extensively both in the textile industry [13] and in the computational geometry community [14]. These methods can be used in our application to obtain the texture atlas; alternatively, pattern pieces can be arranged on the canvas manually. The key difference between our pattern layout and traditional marker planning is that our process does not need to consider matching prints but only the orientation of grain on the pattern pieces. Our process is similar to planning markers on plain fabrics, because the texture is printed later on the patterns.

# 3.4 Texture coordinates assignment

Before prints are designed directly on the 3D garment model, the texture coordinates for all the vertices on the garment should be assigned. The vertices are divided into two categories, including interior points and feature points along the seams (Figure 8). The first category is interior vertices of the 3D patches, and the assignment of texture coordinates of these vertices is straightforward because of one-to-one correspondence between these vertices and those vertices on 2D mesh patterns. The second category consists of the feature points along the seams on the 3D garment. As discussed in Section 3.2, the 3D patches merge at seams (usually the boundary curve of 3D patches) to form a completed garment model. As a result, each vertex in this category corresponds to multiple vertices on the 2D mesh patterns, one on each joining pattern piece. The assignment of texture coordinates for this kind of vertex must be carefully planned to avoid texture mismatch.

![](images/8f2eda35e756f4313ef4597a2c179d7e7d5bb80bbb4b48c30d79bb4090d2a8c9.jpg)

![](images/dc836821fb53b1b7dba1b09f088d13b0b19393b2730cfb5640311813adf4da08.jpg)  
Figure 8. Two categories of vertices. (a) A 2D pattern piece and (b) the corresponding 3D patch of the garment. The interior vertices (grey) belong to the first category; the vertices on the seams (red) are the second category.

A section of a 3D garment is composed of two 3D patches, which correspond to pattern pieces P1 and P2 as shown in Figure 9(a), (b) and (c). Red dots $\{\nu_{l},\nu_{i},\nu_{j},\nu_{k}\}$ are the seam vertices on the garment surface. A seam vertex will appear in all joining patterns, in both P1 and P2 in this case. The triangle $\{\nu_{i},\nu_{j},\nu_{m}\}$ is mapped to P1 and $\{\nu_{i},\nu_{j},\nu_{n}\}$ is mapped to P2. Assigning either $\nu_{i}(u_{1},\nu_{l})$ in P1 or $\nu_{i}(u_{2},\nu_{2})$ in P2 as the texture coordinate for the vertex $\nu_{i}$ on the 3D surface will cause texture mismatch problems in 2D pattern pieces. In other words, if the texture coordinates are not set properly for the seam vertices, the textures mapped on the patterns are not consistent when they are sewn up. For example, the triangles $\{\nu_{i},\nu_{j},\nu_{m}\}$ and $\{\nu_{i},\nu_{j},\nu_{n}\}$ on 3D surface are painted green (Figure 10(a)). If $\nu_{i}$ and $\nu_{j}$ in P1 are assigned as the texture coordinates for the vertices $\nu_{i}$ and $\nu_{j}$ of the garment surface, the corresponding painted result on the 2D patterns is

shown in Figure 10(b). A similar problem is also found if P2 is assigned (Figure 10(c)). Although textures on the two 3D patches are consistent without seams since the print is applied on the garment surface directly, the result on the 2D patterns is not correct. To tackle this problem, we duplicate the seam vertices according to the number of joining patterns. For example, seam vertices $\{\nu_{l},\nu_{i},\nu_{j},\nu_{k}\}$ are the joint of two patterns P1 and P2, and thus $\{\nu_{l},\nu_{i},\nu_{j},\nu_{k}\}$ is duplicated once to obtain $\{\nu_{l}^{\prime},\nu_{i}^{\prime},\nu_{j}^{\prime},\nu_{k}^{\prime}\}$ with the same 3D position but with different texture coordinates in P1 and P2, respectively. The correct result is shown in Figure 10(d).

![](images/955c01fce340c382dde60bd37bb9aafa3db697027b42f2ed4b596e859c713fff.jpg)  
3D Patch   
(a)

![](images/4176725295e2aed9ff685b3df4d11d70963f66907ecf48bfd1d71f5666c15249.jpg)  
P1   
(b)

![](images/e6dba44d0ed477ba55c1e9649821e0203b30ede692671757be93469fb1a3c9df.jpg)  
P2   
(c)   
Figure 9. Texture coordinates for seam vertices (shown in red dots). (a) 3D garment surface. Triangles in grey are mapped to pattern P1 shown in (b) and triangles in blue are mapped to pattern P2 shown in (c).

![](images/b166760824f67ae1a83775b4f17b4da5672334ce5c19a61623a57816f890447e.jpg)  
(a)

![](images/ba320ff44e5ef73a36c02d7fd4f29e0b372083ef54657772b441157441cf1180.jpg)  
(b)

![](images/db547f9bf36c6c6f54c5e19efb0a243ca48757ba5cb04e3a1bb36972b9efb8b4.jpg)

![](images/e13e7ba4de3d772cee2183ded43a0809823b9d5e86fbdc9a866a7bea8879b5a0.jpg)  
(d)

![](images/03baa1ad1d033ee2b5e89f377a535c3443f328cdc0effd834e39971cc2b96f57.jpg)  
Figure 10. (a) 3D surface with two triangles painted green, (b-c) texture on pattern pieces is not consistent if the texture coordinates are not set properly, and (d) texture is consistent on the pattern pieces after the improvement.

# 3.5 Printing designs on a garment

As discussed in Section 2.2, matching patterned fabric on 2D is tedious and time-consuming, and it cannot ensure continuity of texture in all areas (see Table 1). In this paper, we propose to design a textile on a 3D garment model directly by using two techniques, namely 3D digital painting and texture projection. The seams between patches of the garment model are hidden from users, and thus users are allowed to draw freely any figure-flattering designs without the

constraint of seams. The corresponding 2D textured pattern pieces are obtained on the fly because of the proper parameterisation and texture coordinate assignment.

# 3.5.1 3D digital painting

Texture mapping is the most usual way to decorate the surface of 3D objects. Precise texture mapping is very challenging for a complex surface [15]. For instance, decorating a humanoid model will require the texture representing the eyes to be mapped to the actual eyes' position on the model.

3D painting can be regarded as the reverse process of texture mapping. 3D painting allows users to paint directly on the surface of a 3D model. It is a very convenient way to generate desired texture maps. The first direct painting and texturing system, named WYSIWYG, was introduced by Hanrahan and Haeberli [16]. In their method, parameterisation is not involved because the painted colour information is stored at the mesh vertices instead of on a separate texture bitmap. Nowadays, most 3D painting techniques use surface parameterisation to map the painting on the 3D surface onto the corresponding texture map. Therefore, when users paint directly in a 3D view, they are indeed editing the corresponding 2D texture bitmap [17, 35]. We developed a painting system similar to that of [36] in this paper. During painting, when the user paints strokes in the 3D view on object, the system subsequently re-projects the user's strokes to the corresponding position in the 2D texture bitmap based on the predefined UV mapping (texture coordinates). Next, the strokes are displayed on the 3D surface by texture mapping.

# 3.5.2 Texture projection

Painting is not a trivial task, and 3D digital painting is also time-consuming. Even if the user is familiar with 3D digital painting, it still takes hours or days to finish a painting, depending on its complexity. Aside from painting directly on a 3D surface using brushes, an alternative technique for the design of prints on garments is texture projection. This method takes less time and does not require the user to have painting skills.

The direction of projection is important; less texture distortion is produced when the projection conforms more closely to the shape of the 3D object. We propose two schemes of texture projection: cylindrical projection and plane projection.

In the first scheme, we choose cylindrical projection for the global texture design since a human model can be approximated as a cylinder. First of all, the cylinder is geometrically derived from closely fitting a tube around the garment model. The cylinder can be unfolded to define a map of rectangle texture. Based on the relationship between the garment and the cylinder, we project

the texture of this virtual cylinder onto the garment. We obtain the texture-embedded pattern pieces. Since the 3D garment model has relationships with both rectangle texture and embedded pattern pieces, we thus define the relationship between rectangle texture and pattern pieces. Given this relationship and the rectangle texture as the source UV map, we can get the textured pattern pieces as a target map. A video showing the cylindrical texture projection is provided in this paper

In the second scheme, plane projection is used for local texture design. Once the model has been dropped to the canvas, we import the image, rotate it and scale it to the desired position and size on the screen, and then use the projection to paint it onto the model.

# 3.6 Post-processes

# 3.6.1 Seam allowance texture assignment

Pattern pieces without seam allowances are input for the surface parameterisation process (Section 3.2). It means the edges of these pattern pieces are seam lines not cut lines (see Figure 4(c). Since the output patterns with matched texture will be digitally printed, cut and sewn up as a garment, the seam allowance must be added as a post-process. Seam allowances can be added easily by using pattern design software or by simple geometric operations (see Appendix A).

In this section, we assign texture to the seam allowance area. In order to have continuous texture across seams, we first divide the seams into two categories and different schemes of texture assignment are used. The first category of seams is where two or more pieces of fabric are joined. The second category of seams is mainly used in edge treatment, finishing the raw or cut edge of a garment (e.g. the hem of a dress). Figure 11(a) shows one section of the skirt panel; the left and right seams belong to the first category, and the top seam belongs to the second category. For the first category of seams, the texture assignment is as follows. Take Figure 11(b) as an example, where the seam joins panel 1 and panel 2. Let A and B be the seam allowances for panels 1 and 2 respectively and both of them have the same width. To have continuous texture along seams, we first calculate the inner offset parts of C and D in the panels, whereby C and D have the same width as A and B. C and D can be generated as follows:

$$
S _ {\text {i n n e r}} = S + d N \tag {4}
$$

where $S$ indicates any seam curve on the pattern pieces, $N$ is the unit normal of $S$ , and $d$ is the offset distance. The offset distance $d$ is determined by the seam allowance width. Given the current vertex $v_{cur} \in S$ , the vertex $v_{cur}'$ on the $S_{inner}$ can be obtained by

$$
v _ {c u r} ^ {\prime} = v _ {c u r} + d \frac {c}{| c |} \tag {5}
$$

where $c$ is the vector giving the direction of moving $\nu_{cur}$ .

To obtain the texture for C and D, we compute the texture coordinate for each vertex on C and D. Specifically, for each vertex $\nu_{cur}^{\prime}$ on C and D, the first step is to identify the triangle $T$ in which $\nu_{cur}^{\prime}$ is located on the panel. The texture coordinate for $\nu_{cur}^{\prime}$ is the barycentric interpolation of three vertices' texture coordinates in the triangle $T$ .

![](images/cc4b9f7ed30796ccdfafd06a1fd5813d841241ebffce2333afbf24fbef106e3f.jpg)  
(a)

![](images/366374603ba163cc8ba0dfaf3732f3b235ad85401ca8328d3e2d24fbd2c4625d.jpg)  
(b)

![](images/a8461c1d71c56ccf49e50469492671fd640ed08d1c28794626cde5c9d21c2c14.jpg)  
(c)

![](images/8aabc5286914741ee33f43dd364e89a92b5d8b5bbe4dd642985807d3e599fb6d.jpg)  
(d)

![](images/b870be894d1a0f9938392dd37312f0697f419780fbee17dc3fe02249a3107692.jpg)

![](images/e78b9bee257ade2147a475d4e92bebdd179df4ebec91e6fe3d88b4e155e5f6ce.jpg)  
Figure 11. Seam allowance calculation. (a) Sample pattern piece with seam allowance; (b) seam generated by joining two or more different pieces; (c) seam generated by flipping section of the same piece, (d) and (e) are the zoom results of (b) and (c) respectively.

Finally, the texture for seam allowances is obtained by copying the texture of D to A; this is done by assigning the texture coordinate of each vertex on A to that of the corresponding vertex on D. Similarly, the texture of C is copied to B. The zoom result is shown in Figure 11(d).

For the second category of edge seam, A is the seam allowance and B is the inner part obtained by the same method described above (as shown in Figure 11(c) and (e)). The texture for A is obtained by flipping the texture of B, and it can be done by assigning the texture coordinate for

vertices on A to that of the corresponding vertex on B. Figure 11(a) demonstrates the final result of a panel with seam allowance, where the seam allowance is distinguished from the pattern piece by being rendered in a different tone.

# 3.6.2 Drape simulation

In our computer system, a physically based cloth simulation is introduced as another post-process, so that users can visualise the final drape of the garment with texture. In this post-processing step, the draping effect is obtained by colliding the garment with the human model, which is regarded as the rigid object.

In the simulation, the garment is modelled as a particle system, where each vertex of the garment model represents one particle. The particles are connected with forces. Physically-based cloth simulation is formulated as a time-varying partial differential equation, and it can be solved numerically as a differential equation:

$$
\frac {\partial v}{\partial x} = M ^ {- 1} \left(- \frac {\partial E}{\partial x} + F\right) \tag {6}
$$

where $\nu$ is the velocity, $x$ is the position, $M$ is the spring system matrix, $F$ represents the external forces, $E$ represents the internal forces. Cloth dynamics is a complex phenomenon. In our application, stretch forces and bending forces are involved as internal forces to simulate a realistic garment. Given the positions and velocities of particles at time $t$ , we use an implicit Euler integration method to compute the particle positions and velocities at time $t + \Delta t$ .

$$
v (t + \Delta t) = v (t) + M ^ {- 1} f (x + \Delta t) \Delta t \tag {7}
$$

$$
x (t + \Delta t) = x (t) + v (x + \Delta t) \Delta t
$$

where $f$ is the total force exerted on the particle.

Collision detection and response are also essential parts of realistic cloth simulation [23]. The collision is handled during the calculation of implicit Euler integration of positions and velocities. The system detects the collisions between garment and human models, and self-collisions of the garment by using vertex-face, edge-edge, vertex-edge, and vertex-vertex pairs to check whether or not two objects will collide. If the collisions do not intersect at the current time, we enforce a minimum distance between the garment and human models. To deal with the current existing intersection, our algorithm uses a position correction strategy to move two previously collided objects away from each other with a given minimum distance.

# 4. Experimental results and discussions

A total of four garment models are used to demonstrate the pattern textile design method proposed in this paper. Table 2 give details of the garment models and their pattern pieces.

Table 2. Garment model and details.   

<table><tr><td rowspan="2">Garment models</td><td rowspan="2" colspan="2">Number of pattern pieces and details</td><td colspan="2">Print design methods</td></tr><tr><td>3D painting</td><td>Texture projection</td></tr><tr><td>Simple top (with darts)</td><td>4</td><td>front panels × 2; and back panels × 2</td><td>✓</td><td></td></tr><tr><td>Blouse (pattern pieces with varying angles)</td><td>9</td><td>front panel ×1; back panels ×2; side-front panels ×2; side-back panels ×2; and sleeves ×2</td><td>✓</td><td>✓</td></tr><tr><td>Dress (pattern pieces with varying angles)</td><td>10</td><td>front panels × 2; back panels × 2; side-front panels × 2; side-back panels ×2; and sleeves ×2</td><td>✓</td><td>✓</td></tr><tr><td>Skirt (with waistband)</td><td>9</td><td>skirt panels × 8; and waist band × 1</td><td></td><td>✓</td></tr></table>

# 4.1 Results of 3D painting and texture projection

The textures of three garment models including a simple top, a blouse and a dress are designed by 3D painting, and the results are demonstrated in Figure 12, Figure 13 and Figure 14, respectively. The simple top has bust, shoulder and waist darts; the blouse and dress have pattern pieces with varying angles or curved seams. In garments with such types of pattern pieces, continuous texture cannot be achieved by engineered patterns (as discussed in Section 2.2 and Table 1).

Because the garment models are painted by interactive brush painting, completely continuous texture can be achieved as shown in the corresponding textured pattern results in these three figures. Because of the one-to-one texture correspondence established before painting, when the user paints strokes on the object, the corresponding result on the pattern pieces can be seen on the fly. Alternatively, if the user edits 2D pattern pieces, the result is visualised on a 3D garment.

A limitation of 3D painting is that users must have good painting skills. Although a graphic tablet and some editing tools are provided, it still takes a long time to paint a design in 3D. For example, it took almost three hours to complete the vintage floral painting on the simple top garment model (Figure 12(a)). For the irregular geometric pattern on the blouse and the simple stripe pattern on the dress, it took more than an hour and half an hour, respectively. Even though the design is not complex, it follows the contours of the body and flatters the wearer's figure.

![](images/074499b37c136ab41ed70efa2c4481eab1dd34cd02316cce5f0bbd4d2e3e3728.jpg)  
(a)

![](images/c6df14f07de8782a93e819b51f8c251e735941d8a4ad143ec4f55df8c00f86aa.jpg)

![](images/ce96c08765a0662649e7990058eb16533befcddb1064de8fa2dd926d70247253.jpg)  
(b)

![](images/b03ef5c0b781c92f6d32823c8441db83bc6ea3f6ddc6995d585ce08dddfbb33e.jpg)  
Figure 12. 3D painting result for the simple top model. (a) 3D painting result, and (b) the resulting pattern pieces.   
(a)

![](images/9fa07bc9b8a118573fff2c188b49ea4c8da7c2e0ce2e293bc0a577e6fcf77f26.jpg)  
(b)

![](images/cbe4e5e94721304daa53442ae64ce353effebf126783ad96564562d64f731609.jpg)  
(c)

![](images/54574c70ffc96906a176a5b4ca50facfab780737e8ec33c1cd543561df6e7e9e.jpg)  
(d)   
Figure 13. 3D painting result for the blouse model. (a-c) Front, side, back views of the blouse, and (d) the resulting pattern pieces.

![](images/97749f1b8317863848fe518e054f5c70daf4e75e401d2af2892f67038d8f0417.jpg)  
(a)

![](images/a8f7bde44e42b05f2e5a2b810f40dfc3ed311bc75aacb57bbb1ae0e99c390687.jpg)  
(b)

![](images/8f05163da43a4a34dae202bab04b3d1250dc5842bfa043933c0fe40791056ee1.jpg)  
(c)

![](images/f9873189cf922bf1547cd5665d98e37a02989c250c3f4e77fe96d42a36dc53a4.jpg)  
(d)

![](images/4a2a6e64736481056a5d0b2ec3f684d788a19351088af98aff04f00b4c71e1ae.jpg)  
Figure 14. 3D painting result for the dress model. (a-c) front, side, back views of the dress, and (d) the resulting pattern pieces.

Figure 15, Figure 16 and Figure 17 show the results of texture projection for the completed dress, the blouse, and a skirt. Figure 15 shows (a) the input texture, (b) the front and (c) the side views of the dress, and Figure 15(d) shows the resulting pattern pieces. Figure 16 shows (a) the input texture, (b) design results on a blouse and (c) pattern pieces. Figure 17 shows the input texture, (b) front and (c) top views of the projection result on the skirt, and Figure 17(d) is the result on pieces. Because the cylinder is generated by rolling a rectangle texture and joining the two edges of the texture, it is preferred to adopt input texture that supports seamless tiling. For example, the results shown in Figure 16 and Figure 17 are generated by projecting marbling texture [34].

With a human model as a cylinder, results obtained from cylindrical projection have slight texture distortions in the underarm area. Thus, this technique is good for all-round garment types like dresses and skirts, but may not be suitable for garments with long sleeves because the underarm areas will be covered by the sleeves. In this regard, plane projection can be used to design texture in specific regions. More results including plane projection and cylindrical projects can be found in Figure 18, where some results are obtained from regional plane projections (see examples in (b)(c) and (e)).

![](images/da19f07ba1b42540281076d9c7e7407e0a44cb851c8439e7ac562c5a0ddb8133.jpg)  
(a)

![](images/9fbef1ac5ea58b61f3bd2dd79116a1b037647ab2ecb46f58ed1fd55f311fa1f7.jpg)  
(b)

![](images/2a9546a022108e172fa29220c9d3f6ddc96dc924feaddfce6c721b2db493a1fa.jpg)  
(c)

![](images/981972a9f4b5b62e619a45229003cd49a4e576d28187bcee13839c43dde68c6d.jpg)

![](images/3a2335ad15f8e603c88b3e5eef24fde3192890eab1bad691d483e20babba68b1.jpg)

![](images/23b3a547eaba7307bb16ea7a7de3cda3bb37e7c0b7df4db405205f302bf4ccea.jpg)

![](images/6ebd36483a9083faec5eb6b050dc1740f0b8456005d62b7c0ceeff5d558ebd5e.jpg)

![](images/a18bc7832980f75df43e3fffa73713e48c85303ec191b0cbdd7f85d7c8214092.jpg)  
(d)   
Figure 15. The cylindrical texture projection result on the dress model. (a) Original marbling texture (b-c) front and side views of the dress, and (d) the resulting pattern pieces.

![](images/af5bb0b468cfd579d08449ebcdf261005f32a402f1f5ed5709bba40e3c8c03fc.jpg)  
Figure 16. The cylindrical texture projection result of the blouse model. (a) Original texture, (b) front view of the blouse, and (c) the resulting pattern pieces.

![](images/de66c9fe48a3e0978bc9aa648ecb1fc1fcd4cfd0f5362531328617523da5de41.jpg)

![](images/5dcb67e055feb49b28cd36c86add1f9bd19bf9099999824293ce196a39aa765a.jpg)

![](images/99bf021c2af24eb0d0e6920c131f7790a382c1acdba98c6e3ad157e1cb5dfe41.jpg)

![](images/9332a273dd92538928963899421d7829cfbce9845812c98651a423696da4a4ad.jpg)  
Figure 17. The cylindrical texture projection result of the 8-panel skirt model. (a) Original marbling texture, (b-c) front and top views of the skirt, and (d) the result on 8 pieces.

![](images/ac0ae22eab40516c020a628d775f3c456b865ebbcdb6f270fc370f7e5b6fda83.jpg)

![](images/5a723599f6d641c2ad1d56f1a228d3b7513f392bf52fbd8cd36dfd8d58641b0b.jpg)

![](images/588c11c09de526e2c634935cbfafeb2296c4930463d745099b569d04d4eb41b5.jpg)

![](images/2485a7241b432b682afce284c3c7ec6c4d7c2da366c90effed79666362205b70.jpg)

![](images/65d324407c61a1b818c5ab54a103d1770233e73393161e39d6dc3cbf3ac1af2d.jpg)

![](images/2ef827ffac5fd680a086e55df56bb42fedef59feaf5aed1db64c9d6a69f23c8d.jpg)  
(a)

![](images/713e0ca754ec98c257b9ec09f1606471a626c5c97dd2f7c799b50f24a073aee9.jpg)  
(b)

![](images/9c46f4dab67eaa6c2547249a1cf79423d14a99a628740ba13506839f91e852fb.jpg)  
(c)

![](images/4fc5d63558e3de27793b8f36fe32fe09dd7906010451786ae388849c86e9df92.jpg)  
(d)

![](images/b17976d106e788506306734d85e5c39b64788e63dc4ac2d8c2e5121d3c305082.jpg)  
(e)   
Figure 18. More results of texture projection.

It has been demonstrated that both techniques of 3D painting and texture projection can guarantee the prints on the garments are continuous and the texture distortion is slight. Each technique has its own advantages. For example, 3D painting is an interactive technique whereby users are involved in the design of print on the garment. On the other hand, texture projection demands less from users, and it can automatically obtain 3D prints on a garment based on a given plane texture image. The user can choose either one or both to design seamless and figure-flattering prints on fashion products.

# 4.2 Real garment samples and virtual drape simulation

We used the method described in Section 3.6.1 to obtain clothing patterns with seam allowance (see Figure 19). To facilitate the sewing process, notches are added along the seams. The output pattern pieces with texture are then digitally printed, cut and sewn. Real sample garments are prepared, and these real garments are compared with virtual garments in Figure 20. Before real garment production, we can also simulate the drape effects by using the method described in Section 3.6.2. Figure 21 shows the drape results of a dress and a skirt. In the drape simulation, the garment first falls from a horizontal position because of gravity and then collides with the human model.

![](images/577e3bae9098fc9ae979ba61697e6e235d5f80e6b686ab734fb55361935388e1.jpg)  
(a)

![](images/1166655f785e591940c6e98f7cf783a0d1b6f3658128d32da0a46d51aa926779.jpg)  
(b)

![](images/aba8acdadcf43b8be991783e568a330973a92802d6870f1aeb6ed7e90a53c661.jpg)  
(c)

![](images/b12e1202ddc8531bc4bc71294737fb0a75f63db7e587ae339f1c3d1d99b7b8a7.jpg)  
(d)   
Figure 19. Result of generating seam allowance texture (a-b) patterns without seam allowances; (c-d) seam allowance with texture of the pattern in (a-b).

![](images/e99040d3f4194b77f2c09b71c1ff5543cfd617026bf1cddc737b7475428f942b.jpg)

![](images/f2d5937d615e0f83db692c3193467465ca01d1494429d232452a559361550a11.jpg)

![](images/94d66c267bbbc93b780e630b951d217860abaf95f033e73e4af689d9be76f842.jpg)  
(a)

![](images/5a43cb0a770cb8d2eff70eeab250310348f65665a4a7ec7c0babf5d34163cc66.jpg)

![](images/c3ec34d9e720652df759f52370618c368a9365f791f4a7a353eb0aa56db0092a.jpg)  
(b)

![](images/cdebf88ceebfdbfb49571e2ee08854c04637b42ac32a173e6fa0238a3423b1e8.jpg)

![](images/e7ce3eb3783e1d9324496b72d746b5c20883309f3ccac7dd2b2ab82a4d221b97.jpg)  
(c)

![](images/c7ed2b1ad976274458bd871c8a3add3f7dacebaad147d7c456b4aa6fb9daacf1.jpg)

![](images/6117e0c9eb4c32b11c6e37de189f39b6653d4c86fa3b562040a08f4e81f1569a.jpg)  
(d)   
Figure 20. Comparison of real garments and 3D garments with texture design.

![](images/41f74f36cc1862c9d3290ae354022ab52f84cfa34a6f913a41de46ca1dfa2ef6.jpg)

![](images/f1c33022997e58cd6e79887eea4cc618dc1a1fcbd03d1d51aa99f02758785b1b.jpg)

![](images/cd67443508a94f922df07c1414108109196263bc2048ccf7aadff0df1a1d4bdb.jpg)

![](images/0ca417fd9401378ba41d105e9bf75c04465feec43ccca66112fbdb523e3ed832.jpg)  
Figure 21. Virtual drape simulation.

# 5. Conclusion

In this paper, a novel method for print design on garments has been proposed. The design process is started from 3D garments and ended with 2D pattern pieces with matched prints. It is a reverse process of the traditional garment-making process. Two techniques including 3D painting and texture projection have been introduced to help design the prints. The proposed method provides a tool with which to design continuous prints on both garment and pattern pieces. Moreover, it is a useful tool for making a one-of-a-kind garment by designing personalised prints.

The two techniques have been demonstrated to create interesting prints on four garments, comprising a simple top, a blouse, a dress and a skirt. It is important to note that the prints on other garment styles can also be designed by the proposed method. For cylindrical texture mapping, the garment model is flattened by a close-fit uniform cylinder, and the current method will have texture distortion because of occlusion in the underarm area. Since the garment varies from style to style, we will explore in the future other modes of projection, for instance by dividing the garment model into several parts, each applied with a suitable projection mode.

# Appendix A: Adding seam allowance by contour offset generation

Traditional clothing seam allowance can be generated by offsetting the seam curves of a 2D pattern. Given a curve $C$ , the offset curve $C_{out}$ can be simply written as:

$$
C _ {o u t} = C - l N \tag {a1}
$$

where $N$ is the unit normal of $C$ , and $l$ is the offset distance, which is the width of the seam allowance. The width of the seam allowance affects the strength, durability, appearance, and cost of a garment [24]. The width of the seam allowance depends on the stitch type, the fabric and other factors. For any given width of the seam allowance, namely offset distance $l$ , the seam allowance can be calculated.

![](images/19dfa7f5a19de0faff0270f1dadf920b56232c209a876af6a170006d6b6de00e.jpg)  
Figure 22. Illustration of curve offset.

As illustrated in Figure 22, for each vertex on the curve $\nu_{cur} \in C$ , the corresponding offset vertex $\nu_{cur}'$ is calculated as follows. Let the previous and next vertices of $\nu_{cur}$ be represented as $\nu_{pre}$ and $\nu_{next}$ , respectively. Thus, the direction vector $a$ between $\nu_{pre}$ and $\nu_{cur}$ , the direction vector $b$ between $\nu_{next}$ and $\nu_{cur}$ , and the vector $c$ giving the moving direction of $\nu_{cur}$ are calculated by:

$$
a = v _ {p r e} - v _ {c u r}
$$

$$
b = v _ {\text {n e x t}} - v _ {\text {c u r}} \tag {a2}
$$

$$
c = \frac {a}{| a |} + \frac {b}{| b |}
$$

Let $\theta$ be the angel between edges $< v_{cur}, v_{pre}>$ and $< v_{cur}, v_{next}>$ , so we can then calculate the distance $d$ with Eq. a3.

$$
d = \frac {l}{\sin (\theta / 2)} \tag {a3}
$$

Finally, the offset vertex $\nu_{cur}^{\prime}$ for $\nu_{cur}$ is calculated as:

$$
v _ {c u r} ^ {\prime} = v _ {c u r} - d \frac {c}{| c |} \tag {a4}
$$

Therefore, various seam allowances can be generated with different given seam allowance widths (i.e. offset distance) $l$ .

# Acknowledgements

The work described in this paper was supported by a grant from the Research Grants Council of the Hong Kong Special Administrative Region, China (Project No. 5218/13E). This work was also partially supported by The Hong Kong Polytechnic University (grant no. 2015A030401014) and The Innovation and Technology Commission of The Hong Kong Special Administrative Region, China (grant no. ITS/253/15). Shufang Lu was supported by the National Natural Science Foundation of China (Grant no. 61402410) and Zhejiang Provincial Natural Science Foundation of China (Grant no. LQ14F020004).

# References

[1] Gehlhar, M. The Fashion Designer Survival Guide: Start and Run Your Own Fashion Business. Kaplan, 2008.   
[2] Zieman, N. Nancy Zieman's Sewing A to Z: Your Source for Sewing and Quilting Tips and Techniques. Krause Publications Craft, 2011.   
[3] Bowles, M., and Isaac, C. Digital Textile Design. Laurence King, 2009.   
[4] Knox, K. Alexander McQueen: Genius of a Generation, A&C Black, 2010.   
[5] http://en.wikipedia.org/wiki/Mary_Katrantzou, accessed in March 2015.

[6] World Global Style Network, http://www.wgsn.com/fashion/, accessed in July 2016.   
[7] Zhu, S., Mok, P.Y., and Kwok, Y.L. An efficient human model customization method based on orthogonal-view monocular photos. Computer-Aided Design, 45(11): 1314-1332, 2013.   
[8] Wilson, J. Handbook of Textile Design, Woodhead Publishing Ltd, 2001.   
[9] Meng, Y., Mok, P.Y., and Jin, X. Interactive virtual try-on clothing design systems. Computer-Aided Design, 42(4): 310-321, 2010.   
[10] Wang, C.C., Smith, S.S., and Yuen, M.M. Surface flattening based on energy model. Computer-Aided Design, 34 (11): 823-833, 2002.   
[11] McCartney, J., Hinds, B.K. and Seow, B.L. The flattening of triangulated surfaces incorporating darts and gussets. Computer-Aided Design, 31(4): 249-60, 1999.   
[12] Meng, Y., Mok, P.Y., and Jin, X. Computer aided clothing pattern design with 3D editing and pattern alteration. Computer-Aided Design, 44(8): 721-734, 2012.   
[13] Wong, W.K., Wang, X.X., and Mok, P.Y., et al. Solving the two-dimensional irregular objects allocation problems by using a two-stage packing approach. Expert Systems with Applications, 36(2): 3489-3496, 2009.   
[14] Nöll, T., and Strieker, D. Efficient packing of arbitrary shaped charts for automatic texture atlas generation. Computer Graphics Forum, 30(4):1309-1317, 2011.   
[15] Gal, R., Wexler, Y., and Ofek, E., et al. Seamless montage for texturing models. Computer Graphics Forum, 29(2): 479-486, 2010.   
[16] Hanrahan, P., and Haeberli, P. Direct WYSIWYG painting and texturing on 3D shapes. ACM SIGGRAPH Computer Graphics, 24(4): 215-223, 1990.   
[17] Igarashi, T., and Cosgrove, D. Adaptive unwrapping for interactive texture painting. Proceedings of the 2001 Symposium on Interactive 3D graphics, pp. 209-216, 2001.   
[18] Tamstorf, R., Jones, T., and McCormick, S.F. Smoothed aggregation multigrid for cloth simulation. ACM Transactions on Graphics, 34(6), Article no.245, 2015.   
[19] Sigal, L., Mahler, M., Diaz, S., McIntosh, K., Carter, E., Richards, T., and Hodgins, J. A perceptual control space for garment simulation. ACM Transactions on Graphics, 34(4), Article no.117, 2015.   
[20] Chen, X., Zhou, B., Lu, F., Wang, L., Bi, L., and Tan, P. Garment modeling with a depth camera. ACM Transactions on Graphics, 34(6), Article no.203, 2015.   
[21] Berthouzoz, F., Garg, A., Kaufman, D.M., Grinspun, E., and Agrawala, M. Parsing sewing patterns into 3D garments. ACM Transactions on Graphics, 32(4), Article no.85, 2013.   
[22] Bartle, A., Sheffer, A., Kim, V.G., Kaufman, D.M., Vining, N., and Berthouzoz, F. Physics-driven pattern adjustment for direct 3D garment editing. ACM Transactions on Graphics, 35(4), to appear, 2016.   
[23] Tang, M., Wang, H., Tang, L., Tong, R., and Manocha, D. CAMA: contact - aware matrix assembly with unified collision handling for GPU - based cloth simulation. Computer Graphics Forum, 35(2): 511-521, 2016.   
[24] Brown, P. and Rice, J. Ready-to-wear Apparel Analysis, $3^{\mathrm{rd}}$ Edition, Prentice-Hall Inc., 2001.   
[25] Kang, T.J., and Kim, S.M. Development of three-dimensional apparel CAD system part 1: Flat garment pattern drafting system. International Journal of Clothing Science and Technology, 12(1): 26-38, 2000.   
[26] Hu, Z., Ding, Y., Zhang, W., and Yan, Q. An interactive co-evolutionary CAD system for garment pattern design. Computer-Aided Design, 40(12): 1094-1104, 2008.   
[27] Chen, Y., Zeng, X., Happiette, M., Bruniaux, P., Ng, R., and Yu, W. Optimisation of garment design using fuzzy logic and sensory evaluation techniques. Engineering Applications of Artificial Intelligence, 22(2): 272-282, 2009.   
[28] Lu, J.M., Wang, M. J.J., Chen, C.W., and Wu, J.H. The development of an intelligent system for customized clothing making. Expert Systems with Applications, 37(1): 799-803, 2010.

[29] Yang, Z.T., Zhang, W.Y., Zhang, W.B., and Xia., M. The research on individual clothing pattern automatic making technology. International Conference on Artificial Reality and Teexistence--Workshops, pp. 453-457, 2006.   
[30] Okabe, H., Imaoka, H., Tomiha, T., and Niwaya, H. Three dimensional apparel CAD system. Proceedings of the 19th Annual Conference on Computer Graphics and Interactive Techniques, 26(2): 105-110, 1992.   
[31] Fuhrmann, A., Groß, C., Luckas, V., and Weber, A. Interaction-free dressing of virtual humans. Computers and Graphics, 27(1): 71-82, 2003.   
[32] Igarashi, T. and Hughes, J.F. Clothing manipulation. ACM SIGGRAPH 2006 Courses, p21. San Diego CA, 2006.   
[33] Satam, D., Liu, Y., and Lee, H.J. Intelligent design systems for apparel mass customization. The Journal of the Textile Institute, 102(4): 353-365, 2011.   
[34] Lu, S., Mok, P.Y., and Jin, X. From design methodology to evolutionary design: An interactive creation of marble-like textile patterns. Engineering Applications of Artificial Intelligence, 32: 124-135, 2014.   
[35] Fu, C.W., Xia, J., He, Y. Layerpaint: a multi-layer interactive 3D painting interface. Proceedings of the SIGCHI Conference on Human Factors in Computing Systems. ACM, pp. 811-820., 2010.   
[36] Maxon Body Paint 3D, http://www.maxon.net/en/products/bodiespaint-3d/   
[37] Xu, J., Mok, P.Y., Yuen, C.W.M., and Yee, R.W.Y. A web-based design support system for fashion technical sketches. International Journal of Clothing Science and Technology, 28(1): 130-160, 2016.   
[38] Mok, P.Y., Xu, J. and Wu, Y.Y. Fashion design using evolutionary algorithms and fuzzy set theory – a case to realize skirt design customizations, In Tsan-Ming Choi (Ed.) Information Systems for the Fashion and Apparel Industry, Chapter 9, pp. 163-198, Woodhead Publishing, United Kingdom, 2016.