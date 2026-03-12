<!-- markdownlint-disable -->
# AI-assisted Pattern Generator for Garment Design

Michael Danner $^{1,*}$ , Elena Alida Brake $^{2}$ , Gabriela Kosel $^{2}$ , Yordan Kyosev $^{3}$ , Katerina Rose $^{2}$ , Matthias Ratsch $^{2}$ , Holger Cebulla $^{4}$

1YAHATA GmbH, Starzach, Germany   
$^{2}$ Reutlingen Research Institute, Reutlingen University, Reutlingen, Germany   
$^{3}$ TU Dresden, Institute of Textile Machinery and High-Performance Material Technology, Chair of Development and Assembly of Textile Products, Dresden, Germany   
4TU Chemnitz, Institute of Lightweight Structures, Chair of Textile Technology, Chemnitz, Germany   
*Corresponding author E-mail address: info@yahata.de

# INFO

CDATP, ISSN 2701-939X

Peer reviewed article

2024, Vol. 5, No. 2, pp. 195-206

DOI 10.25367/cdatp.2024.5.p195-206

Received: 30 March 2024

Accepted: 27 July 2024

Available online: 29 December 2024

# ABSTRACT

This paper introduces an AI-assisted pattern generator, aimed to simplify garment design by flattening the pattern creation in an automated process from 3D scans for users without knowledge of conventional pattern construction. This garment tool plug-in computerizes the development of scanned persons into 3D shell surface meshes, which are automatically unwrapped into 2D patterns, streamlining the traditionally complex aspects of garment design for novices. The process uses advanced AI algorithms to facilitate the conversion of 3D scans into usable patterns. Machine learning adapts to different garment styles (close-fitting, regular fit and loose-fitting), ensuring a broad applicability, while customization options allow a precise adaption to individual body measurements. This AI-assisted tool enables a wider audience to generate customized garment creation.

# Keywords

Al-assisted pattern construction,   
pattern generation,   
CAD flattening,   
machine Learning

© 2024 The authors. Published by CDATP.

This is an open access article under the CC BY license https://creativecommons.org/licenses/ peer-review under responsibility of the scientific committee of the CDATP.

© 2024 CDATP. All rights reserved.

# 1 Introduction

Digital CAD (Computer Aided Design) tools are very important for the clothing technology and pattern development. Not only have they revolutionised the garment manufacturing process, but they are also helping to increase efficiency, precision and creativity in the fashion industry. A key aspect is the precision that digital CAD tools bring to the pattern development process. By using precise digital measurements and software applications, designers and technicians can develop accurate patterns that fit the human body correctly. In addition, digital CAD tools enable seamless collaboration between

different players within the textile chain. Designers, pattern makers, tailors and manufacturers can work on a digital model in real time, which improves communication and shortens time to market. Another advantage is digital CAD tools help to reduce resource consumption and environmental impact. By avoiding physical prototypes and using virtual simulations, companies can minimize material waste and production waste, which contributes to more sustainable apparel production. Overall, digital CAD tools play a crucial role in modernizing and optimizing the garment technology and pattern development. They enable a precise fit, improve collaboration in the production chain, speed up the design process and contribute to the sustainability of the fashion industry. Therefore, they are indispensable tools for fashion designers and garment technicians [1]. At time, when the fashion industry is shifting to digital and content production as well as marketing activities are becoming more virtual, the importance of digital tools and technologies is becoming increasingly apparent. For gaming and 3D artists in particular as well as fashion companies, it is becoming essential to develop efficient methods of creating virtual garment renderings that are both engaging and accurate [2].

While digital CAD tools are already a major advancement in the garment engineering and pattern development, it is equally important to create options that people without formal training or experience in the field can use. For this reason, we present a new approach: Our objective is to develop an AI-assisted pattern generator that enables people to create virtually basic patterns for garments, even without specific knowledge or experience in pattern construction. By integrating AI-technologies and algorithms, users can import their own 3D scans or the models they want to work with to generate an ideal construction body (ICB) and create basic patterns for tops in this first draft. Our AI-assisted pattern generator represents the first step in making pattern construction accessible to everyone and breaking down the barriers between the traditional craftsmanship and digital innovation.

# 2 Theoretical foundations

The traditional approach to pattern construction in garment engineering is based on a craft process that requires a variety of specialized skills and techniques. Typically, this process begins with the creation of basic patterns, which serves as a starting point for the construction of various garments. To create a basic pattern, body measurements are first taken and are used to calculate the construction values. These measurements form the basis for creating a two-dimensional pattern, which is brought to the three-dimensional body shape. This basic pattern is then adjusted and refined on a model or mannequin to ensure that the pattern fits, and the desired fit is achieved. This iterative process of adjustment and revision may require several iterations until the desired result is achieved. Traditional pattern construction requires a high level of expertise, experience and craftsmanship to create precise and accurate patterns [3].

# 2.1 Conventional Pattern Construction Methods

There are many different pattern systems with different approaches to construction development, each with their own advantages and target groups. The selection of system often depends on the specific requirements, preferences and resources of the application:

- M. Müller & Sohn pattern system is most widely used in the international clothing industry and was developed by Michael Müller in Munich back in the $19^{\text{th}}$ century. Since then, it has been continuously optimized and adapted to the current industry requirements. It is divided into the areas of women, men and children. This construction is also based on the creation of a measurement set which is generated using body measurements and construction formulas [15].   
- Hohenstein: Like the M. Müller & Sohn design system, the Hohenstein pattern system is used to create basic and model constructions based on dimension sets. However, here the model cuts are designed according to the later visual appearance, i.e. the silhouettes [16].   
- The Optikon system is an androgynous process concept developed by the Niederrhein University of Applied Sciences as part of a research project and is also used in the industry [14]. The so-called closed system Optikon is suitable for the construction of entirety of garments for both women's and men's outerwear [6]. The primary body measurements are necessary for the

design: chest circumference, body height, buttock circumference, waist circumference, and figure type. In addition, the secondary body measurements are calculated using formulas. These formulas are based on the correlation of primary and secondary measurements defined in the standard measurement tables and are used to construct garments with individual measurements. The construction-related coordinate system is created from the primary and secondary body measurements, which shows the relationship between the human body to be enveloped and the individual flat parts that will envelop the body. Here, both the human proportions and clothing-specific parameters for the construction of models are taken into account [6,14].

# 2.2 Flattening

Flattening, also known as unwinding, represents a fundamental methodology in garment pattern development. This process technique aims to transform a three-dimensional garment into a two-dimensional shape that is used as a pattern. Flattening plays a central role in the realization of designs from the concept to production by enabling designers and pattern makers to create accurate and scaled patterns. By applying mathematical principles and geometric calculations, complex three-dimensional shapes and curves can be projected onto simplified, flat surfaces without distortion. This approach forms the basis for the production of accurately fitting garments, with flattening acting as an indispensable technique [4,5].

Dividing the upper torso into eight regions is the most precise method for processing a body scan, as shown in Fig. 1. The lower half of the upper torso can be unrolled by adding two further regions. This results in a total of 12 regions, which give 24 regions for the right and left halves of a body. The chest point and waist line, as well as the front and back center, provide the reference points for classifying the torso. This method is used to unwind the three-dimensional body because, for example, it is not possible to unwind an entire front section in one piece without distortion. This is due to the fact that darts are used in the two-dimensional pattern construction to reshape a human body shape. With this method, the regions are divided where darts would be placed in a construction. First, the bust point, the center front and center back and the side seam are determined on the 3D scan. The height of the waist and hip line is then defined and the torso can be divided into 12 / 24 regions. Surface flattening was selected as the method for pattern creation, demonstrating a notably enhanced precision in fit, particularly in the development of patterns based on individual body data. Traditional pattern construction typically necessitates multiple adjustment iterations to attain the desired fit, resulting in inefficiencies, particularly in the context of individualized or bespoke patterns. Moreover, the quality of pattern components frequently relies on the specific skills of pattern maker. Consequently, conventional pattern construction encompasses numerous interdependent variables like ease based on experience that hinder the assurance of optimal fit across all body types [17].

![](images/e4946f32936bb71677c965fd8a0d5239b19413e7df44500c512e872108b75637.jpg)  
Fig. 1 Flattening torso division.

Surface flattening provides a dependable method to achieve the required garment fit [7]. However, inherent sources of error exist in this approach. When utilizing the body surface directly from scanned data for pattern creation, essential allowances are omitted, impacting both the comfort and the

recognizable garment aesthetics. In addition, the lack of allowances means that only close-fitting garments made of highly elastic material can be covered.

The following criteria must be considered when creating optimized 3D models for flattening to obtain optimal garment patterns:

1 The silhouette of the desired garment - X, O or A.   
2 The position or layer of desired garment - $1^{\mathrm{st}}$ , $2^{\mathrm{nd}}$ or $3^{\mathrm{rd}}$ layer.   
3 The fit of the desired garment - close-fitting, regular fit, or loose-fitting.

The silhouette of the desired garment is important for the initial spacing of garment on certain parts of the body. For example, the distance to the waist is set higher for an O-silhouette than for an X-silhouette. Table 1 shows an example of how offsets for different silhouettes can be classified.

Table 1. Offsets for different silhouettes.   

<table><tr><td>Silhouette</td><td>X-silhouette</td><td>O-silhouette</td><td>A-silhouette</td></tr><tr><td>Offset bust</td><td>Low</td><td>Medium</td><td>Low</td></tr><tr><td>Offset waist</td><td>Low</td><td>High</td><td>Medium</td></tr><tr><td>Offset hip</td><td>Medium</td><td>Medium</td><td>High</td></tr><tr><td>Offset knee</td><td>Medium</td><td>Low</td><td>High</td></tr></table>

The term "layer or position of the desired garment" refers to the various strata of attire worn in succession. For instance, undergarments, which are worn closest to the body, necessitate minimal additional breadth (0 degrees). Conversely, outerwear like blouses $1^{\text{st}}$ layer, jackets $2^{\text{nd}}$ layer, or coats $3^{\text{rd}}$ layer demand a more substantial allowance. These allowances can be derived from traditional pattern drafting methods. Moreover, it is essential to consider the desired fit of the garment. Opting for a slim fit implies that there will be less need for alterations in the body measurements, whereas choosing an oversized fit will demand correspondingly greater allowances in terms of proportions. The initial body shapes utilized were derived from X-silhouettes, tailored in a slim fit for the primary, secondary, and tertiary garment layers without taking its structure into account. Table 2 presents the body dimensions of the original body form employed alongside the respective dimensions of ideal construction bodies. It is crucial to emphasize that any displacement of values along the vertical axis must be avoided. Such a shift would lead to inaccurate positioning of the chest, waist, and hip measurements, consequently leading to an improper fit. Therefore, the offset for vertical axis values must be zero. Table 2 defines the dimensions for different clothing layers [7] according to [8].

Table 2. Dimensions for different clothing layers [8].   

<table><tr><td>Measurement in cm</td><td>Size 38</td><td>Offset</td><td>1stlayer</td><td>Offset</td><td>2ndlayer</td><td>Offset</td><td>3rdlayer</td></tr><tr><td>Body height</td><td>168.0</td><td>0</td><td>168.0</td><td>0</td><td>168.0</td><td>0</td><td>168.0</td></tr><tr><td>Chest circumference</td><td>88.0</td><td>4</td><td>92.0</td><td>5</td><td>97.0</td><td>5</td><td>102.0</td></tr><tr><td>Waist circumference</td><td>72.0</td><td>4</td><td>76.0</td><td>5</td><td>81.0</td><td>5</td><td>86.0</td></tr><tr><td>Hip circumference</td><td>97.0</td><td>4</td><td>101.0</td><td>5</td><td>106.0</td><td>5</td><td>111.0</td></tr><tr><td>Shoulder width</td><td>12.7</td><td>0</td><td>12.7</td><td>1.5</td><td>14.2</td><td>0.5</td><td>14.7</td></tr></table>

# 2.3 Overview of existing digital solutions in garment design

Existing digital garment development programs offer a variety of features and tools to support the entire production process from design conception to manufacturing. Some of the best-known programs in this area are:

- Optitex: Optitex is a software solution for the apparel industry that offers a wide range of features including pattern making, 3D visualization, sizing and virtual fitting. The software enables designers and technicians to create realistic 3D models of garments and develop virtual prototypes [18].   
- Assyst / Style3D: The CAD system Assyst is a solution for the apparel technology, encompassing both 2D and 3D functionalities. Assyst's 2D program allows for precise pattern drafting and a construction development, accommodating various designs and sizes inclusive grading. Integrated with Style3D, a 3D simulation program, it offers the capability to visualize and modeling these patterns within a three-dimensional environment. Style3D enables designers to virtually try on garments, simulate materials and colors, and generate renderings. The combination of Assyst's CAD system and Style3D offers integration between 2D and 3D design processes, leading to more efficient and accurate apparel development [19].   
- Gerber: Gerber Technology offers a suite of apparel manufacturing software solutions, including pattern making, sizing, marker making and the production management. Lectra Modaris is from the same company an important provider of software solutions for the apparel industry. Lectra software includes modules for the design, pattern making, production planning and control, and automated cutting technologies [20].   
- Browzwear: Browzwear specializes in VStitcher 3D design software for the apparel industry. Browzwear's software enables designers to create realistic 3D models of garments and perform virtual try-ons to check fit and design before the production. The corresponding 2D parametric pattern construction software GRAFIS belongs to the VStitcher 3D CAD program [21].   
- CLO: CLO is a 2D and 3D design software for the apparel industry that enables designers to create realistic 3D models of garments and perform virtual try-ons. The software offers a variety of tools for pattern making, sizing and textile simulation to cover the entire production process [22].

These programs offer a variety of functions and tools to support the entire production process from the design concept to manufacturing and have a significant impact on the efficiency, precision and creativity in the fashion industry. Nevertheless, all of them require special knowledge and experience in pattern construction.

# 2.4 Gap analysis

The described advances in the digital garment technology have made it easier to produce patterns from 3D scans. Traditional pattern construction methods are being supplemented by technologies to automate and simplify the pattern making process compared to manually constructed patterns where a fit analysis can only be performed on the physical prototype. A key aspect of evaluating current technologies is the ease of use. Traditional pattern construction methods often require extensive expertise and experience to create precise and customized patterns. The development of an AI-assisted pattern generator with the integration of automation technologies enables users to quickly and efficiently create customized patterns. Even if AI cannot replace experience in pattern making for the various different body shape types and applicable textile materials, different bespoke basic patterns can also be generated by nonprofessionals without having to rely on complex processes. Another important factor is the accessibility of technologies. While traditional pattern construction methods often require expensive and specialized equipment, and modern digital solutions, in the most cases, are only useable through the pricey acquisition of licenses and only run on PCs that fulfill certain technical requirements, the open-source software Blender for our tool is for free and widely available. The availability of software solutions that run on standard hardware platforms and appeal to a wide range of users is helping to increase the accessibility of patterning technologies for the use in various industries. In addition, the development

focuses on automating the process of generating made to measure basic pattern from 3D scans. Through the use of algorithms and artificial intelligence, complex pattern design tasks are automated, resulting in significant time savings and increased efficiency. This enables users to quickly and precisely generate customized patterns without the need for extensive manual intervention.

In this context, measurement programs, 2D pattern systems and 3D simulation and visualization software are crucial components of the workflow. The integration of these technologies enables users to go through a seamless process from data acquisition and pattern creation to virtual fitting and visualization. The advantage is that users, even without extensive expertise, can quickly and easily create customized basic patterns from which they can further develop their designs. For this process of flattening the pattern of the 3D object, the UVs of the scan will be cut in Blender: Blender's UV Editor provides a Shader specifically designed for visualizing UV stretching based on area metrics. This Shader utilizes a heatmap color scheme, ranging from blue to yellow, to represent the degree of stretching across UV faces in the UV map. Consequently, some instances of UV stretching may be challenging to identify, particularly during the model manipulation in the 3D Viewport. In contrast, the UV Editor does offer a feature for selecting all overlapping UV faces, which can be observed in both the UV Editor and the 3D Viewport. Furthermore, Blender has the option to color individual faces of the model based on the orientations of their normals within the 3D Viewport. However, this functionality is confined to the 3D Viewport and relies on the computation of colors from the normal vectors of faces in the 3D model. Notably, there is currently no provision for computing colors based on UV normals, thus restricting the application of this feature to surface normals only. Finally, to visualize any texture on a model, Blender users must manually create a new material, associate the texture with it, and assign the material to the model. This multi-step process is necessary for rendering textures accurately on the model's surfaces within the 3D Viewport and during final rendering [23].

# 2.5 The role of AI in transforming fashion design processes

The integration of Artificial Intelligence (AI) technologies has catalyzed a profound transformation within the fashion design domain, revolutionizing traditional processes and methodologies. AI's role in the fashion design encompasses a myriad of functionalities, ranging from trend forecasting and virtual prototyping to personalized styling recommendations and sustainable practices. By leveraging machine learning algorithms, AI systems can analyze vast datasets of historical trends, consumer preferences, and market dynamics to generate predictive insights, thereby empowering designers with valuable foresight into emerging styles and consumer demands. Additionally, AI-driven virtual prototyping tools facilitate rapid iteration and refinement of designs, enabling designers to visualize and evaluate their creations in a virtual environment before physical production. Moreover, AI-powered styling platforms leverage advanced image recognition and recommendation algorithms to curate personalized fashion suggestions tailored to individual preferences and body types. Furthermore, AI technologies play a pivotal role in promoting sustainability within the fashion industry by optimizing supply chain management, reducing waste through predictive demand forecasting, and facilitating the development of eco-friendly materials and production techniques. Overall, AI represents a transformative force in the fashion design, fostering innovation, efficiency, and sustainability across the entire design process, from ideation to the consumer engagement [24].

In this paper, we used geometric models of garments provided by the Berkeley Garment Library, a resource developed by the UC Berkeley Computer Graphics Research group. These models, designed for the use in cloth simulation, have been used as training data for our network. The Berkeley Garment Library offers a diverse collection of garment models that are essential for the realistic simulation of cloth behavior in various contexts [9,10]. To augment the training data, we used the Sewfactory Dataset [11] which consists of around one million images and ground-truth sewing patterns for model training and quantitative evaluation. Additionally, we used the "Dataset of 3D Garments with Sewing Patterns" [12], which consists of 23,500 3D garment models with their corresponding sewing patterns in 12 groups of garment types. Garment Tool 2.0 is a Blender add-on designed to enhance the garment creation workflow. It provides an interface for designing realistic clothing by simulating fabrics, patterns, and stitches. Blender's physics engine is used for the accurate cloth simulation [23].

# 3 Development of the Al-Assisted Pattern Generator Plug-in

![](images/0da4ca4b7b4c748e6e0fd4b01d256cc49d86c1bc16d7043e2031666d1d8eca69.jpg)  
Fig. 2 Workflow AI-assisted pattern generator.

# 3.1 Conceptual framework and objectives of the plug-in

A central part of our project's technology stack is a specially developed tool for Blender. The script was developed with the aim of providing an intuitive, efficient and seamless bridge between the theoretical aspects of our methodology and its practical application. Blender supports easy import of 3D scanned avatars and the tool automates the conversion of raw scan data into a format that can be used for further steps. This script then offers the selection and configuration of clothing parameters. The interface allows users to select the type of garment, adjust fit preferences (e.g. loose, regular or tight fit), and set other customization options, with these parameters seamlessly integrated into the 3D body model.

The determination of seamlines, pattern details (e.g., cut lines, gathers, darts, pockets), and other construction elements is driven by the AI algorithms and pre-set garment templates. Here is how the process is handled:

- Garment type selection: When a user selects the type of garment, the script references a database of predefined garment templates. Each template includes standard seamlines, cut lines, and structural elements typical for that garment type.   
- Fit preferences and customization: Adjusting fit preferences (loose, regular, tight) influences how the templates are adapted. The AI algorithm modifies seamlines and cut lines to accommodate the selected fit. For example, a tight fit may require additional darts to ensure proper shaping, while a loose fit might reduce the number of darts and gathers.

# - Pattern detail generation:

- The number and placement of seamlines are determined by both the garment template and the fit adjustments. The script uses body measurements and fit preferences to position seams in a way that maintains garment integrity and aesthetic.   
- Darts are generated based on the contours of the 3D body model. The AI identifies areas where shaping is necessary and places darts accordingly. For example, bust darts are added for garments that require shaping around the chest area.   
- Pattern lines are based on the garment template and modified by fit preferences. Gathers are added in areas requiring additional volume, such as around the waist for a gathered skirt.   
- Customizable options like pockets are included based on user selection. The script positions these features based on standard garment construction principles and user-defined parameters.

The integration of these details ensures that the generated patterns are comprehensive and ready for production. The analyzed body surface provides the essential data needed to create an ICB 3D body shape for garment fit. The development of this Blender tool represents an initial step in improving the production of customized garments. By integrating complex 3D scanning and garment fitting processes into a familiar and powerful 3D modeling environment, the tool aims to increase efficiency, accuracy and significantly increase the creativity of digital clothing design without explicit garment construction knowledge.

# 3.2 Description of the neuronal network

Another important goal is to integrate the script into the pre-trained neural network that is responsible for generating patterns. This functionality allows garment patterns to be generated and edited directly in Blender, streamlining the workflow from design to pattern creation (Fig. 3).

We employ NeuralTailor [13] as a base model and adapt this model to transform our optimal body surface 3D model into precise garment patterns. The network operates as follows:

![](images/30c6b01214bc5e2161a3f2cb41dc462c24334d70d17eb47c92cd2294edd2559b.jpg)  
Fig. 3 AI-driven garment generation network: 3D model and clothing type parameters are input into an EdgeConvEncoder, creating a latent space. Multiple LSTMs decode a flattened, unwrapped sewing pattern.

- Input processing: The network accepts scans to transform these into an ICB model through the offset values as input. This model represents the detailed topography of an individual's body, providing the essential foundation for custom garment creation.   
- EdgeConv block encoder: The core of the network employs an encoder based on EdgeConv blocks, a variant of convolutional neural networks designed to capture the geometric structure of data efficiently. The encoder processes the 3D model, extracting relevant features and patterns critical for garment fitting. A skip connection is employed from the input to the output of the last EdgeConv layer, enhancing the network's ability to preserve intricate details by directly propagating information from the input to deeper layers in the architecture.   
- LSTM for panel encodings: Following feature extraction, a Long Short-Term Memory (LSTM) network processes the garment latent code. The LSTM, renowned for its effectiveness in handling sequences, generates a sequence of latent vectors corresponding to the different panels of the garment. This step encodes the temporal relationships between panels, ensuring the garment is coherent and structurally sound, when assembled.   
- Panel decoder for shape and stitching information recovery: Finally, a Panel Decoder utilizes the sequence of latent vectors to recover the precise shape and stitching information of each garment panel. This decoder is pivotal in translating the abstract representations back into detailed, actionable sewing patterns, including the shape, size, and how each panel should be stitched together.

This AI network architecture facilitates a seamless transition from 3D body scans to ready-to-use sewing patterns, embodying a significant leap forward in automated, made to measure garment manufacturing.

# 3.3 Features and Functionality

The developed tool processes 3D scans to 3D Ideal Construction Bodies for automatic unwrapping into 2D patterns. The initial step involves generating 3D scans into 3D ICBs, which serve as the foundation for a garment construction. Utilizing AI algorithms, the scans are processed to generate ICB representations. These ICBs are based on the specific measurements of the individual, accounting for variations in body shapes and proportions. Offset values, as per the corresponding Table 2, are then applied at key points such as the bust, waist, and hips circumferences to accommodate different garment layers, ensuring accurate and anatomically correct constructions.

Once the Ideal Construction Body has been generated from the scan, the process transitions to automatically unwrapping the 3D ICB mesh into 2D patterns with the least possible distortion. This intricate process involves unwinding the 3D mesh representation into a flat, two-dimensional pattern, effectively creating the basic pattern for the garment assembly. A user-friendly process, from importing 3D scans to customizing garment styles and sizes. One of the key highlights of the tool is its extensive customization potential for different garment styles and individual adjustments. Users can effortlessly construct garments to suit various style preferences, including close-fitting, regular fit, and loose-fitting designs. In essence, the features and functionality of the garment design tool offer a comprehensive and user-centric approach to garment construction, leveraging advanced technologies to streamline the design process and enhance creative possibilities. From transforming 3D scans into 3D ICB meshes to facilitating customization for different styles and sizes, the tool empowers users to realize their design with a precision and efficiency.

# 4 Application and use cases

This section describes the comprehensive workflow designed to improve garment fitting through advanced 3D scanning and machine learning technologies. The project basis is its ability to adapt clothing design to everyone's unique body shape, improving both comfort and style. The Blender tool works through the sequential steps described below from the first scanning input of a person to the final check of the garment fit.

Step 1: The process begins with a 3D scan of a person. This step involves capturing an accurate three-dimensional representation of the person's body, which serves as baseline data for the subsequent process of customizing the garment.

Step 2: Once the 3D scan is completed, the next step is to select the garment type and configure specific parameters, such as the desired fit layer (loose, regular or tight). This customization allows for a personalized approach to garment creation that caters to the individual's preferences and body shape.

Step 3: After setting the garment type and parameters, the system analyzes the 3D body scan, focusing on the body surface area and relevant measurements. This analysis is critical to understand the unique contours and dimensions of person's body and serves as the basis for the garment's design process.

Step 4: Based on the analysis from the previous step, the workflow generates an ICB model that matches the selected garment parameters. This model acts as a virtual mannequin, ensuring that the designed garment achieves the desired fit.

Step 5: The next step is to retrieve specific sewing data essential for creating the garment's pattern. These data include information about seams, seam allowances, and other essential details necessary for accurate pattern creation.

Step 6: Using a pre-trained neural network, the system predicts the number and shape of different pattern pieces required for the garment. This innovative approach allows the creation of precise, individual, nearly distortion-free pattern for the previously created 3D model.

Step 7: Finally, the workflow checks the garment's fit using 3D simulation. This simulation integrates the scanned 3D model of the person with the newly generated patterns, providing a virtual changing room experience. Any necessary fit adjustments are identified and corrected at this stage to ensure the final garment meets the desired specifications.

# 5 Experiments

For evaluation, we started with a systematic investigation to assess the accuracy and effectiveness of the developed tool in converting 3D scans into ICBs into bespoke garment patterns. The overall goal of these experiments is to evaluate the accuracy of the AI-generated patterns compared to the manually unwrapped patterns using a quantitative analysis approach. A program was developed specifically for this experiment to facilitate this evaluation. The experiment assesses three key metrics: perimeter, area

and HU moments. HU moments are statistical measures used to describe shapes and structures in digital images. They are based on the calculation of higher-order moments of the distribution of pixel intensities in the image and are used to quantify characteristic features such as symmetry. These metrics are evaluated using a Python program and serve as objective measures for evaluating the agreement and congruence between our AI patterns compared to the ground truth patterns based on the following formula:

$$
D (A, B) = \sum_ {i = 0} ^ {6} \frac {\left| H _ {i} ^ {A} - H _ {i} ^ {B} \right|}{\left| H _ {i} ^ {A} \right|} \tag {1}
$$

![](images/f5271ff49043139d4bc88b8a9fa9a1fe95ca2ce603499a7fbb7c0ff2aa6493eb.jpg)

![](images/77365f857c490b3282503343a9966fcc4db320d647f94a917523805a21f603df.jpg)  
Fig. 4 Evaluation of generated AI patterns against ground truth counterparts to automatically assess the accuracy of our experiments.

As shown in Figure 4, the differences of all 7 HU moments are calculated for each pattern piece, then the difference between the two shapes is cumulated as an amount, and divided by the original HU moment of the initial ground truth shape. All pattern pieces of the basic pattern are automatically numbered by the program and assigned to each other by the user. The HU moments are calculated independently of rotation and transformation and provide a measure of how closely the shapes match. The Python program examines area, perimeter and HU moments per section and in total per section, here called pattern number, as shown in Table 3:

Table 3. Experiment results for three selected pattern off-sets.   

<table><tr><td colspan="4">Basic pattern for 1st layer</td><td colspan="4">Basic pattern for 2nd layer</td><td colspan="4">Basic pattern for 3rd layer</td></tr><tr><td>Pattern number</td><td>Accuracy Area in %</td><td>Perimeter in %</td><td>Distance HU moments</td><td>Pattern number</td><td>Accuracy Area in %</td><td>Perimeter in %</td><td>Distance HU moments</td><td>Pattern number</td><td>Accuracy Area in %</td><td>Perimeter in %</td><td>Distance HU moments</td></tr><tr><td>1</td><td>79.1</td><td>98.9</td><td>0.60</td><td>1</td><td>74.5</td><td>97.2</td><td>1.03</td><td>1</td><td>77.7</td><td>95.1</td><td>3.21</td></tr><tr><td>2</td><td>119.4</td><td>99.6</td><td>67.61</td><td>2</td><td>74.3</td><td>96.1</td><td>1.05</td><td>2</td><td>88.9</td><td>99.2</td><td>1.25</td></tr><tr><td>3</td><td>113.0</td><td>100.0</td><td>24.27</td><td>3</td><td>86.5</td><td>98.8</td><td>2.66</td><td>3</td><td>79.6</td><td>93.4</td><td>0.40</td></tr><tr><td>4</td><td>86.0</td><td>98.7</td><td>0.57</td><td>4</td><td>91.2</td><td>96.9</td><td>1.16</td><td>4</td><td>98.0</td><td>99.9</td><td>0.65</td></tr><tr><td>5</td><td>75.0</td><td>94.5</td><td>0.59</td><td>5</td><td>80.2</td><td>91.8</td><td>0.66</td><td>5</td><td>97.9</td><td>97.8</td><td>0.46</td></tr><tr><td>6</td><td>74.8</td><td>93.7</td><td>0.59</td><td>6</td><td>68.8</td><td>94.4</td><td>0.55</td><td>6</td><td>98.3</td><td>99.6</td><td>0.68</td></tr><tr><td>7</td><td>105.8</td><td>100.9</td><td>1.17</td><td>7</td><td>84.0</td><td>99.3</td><td>3.15</td><td>7</td><td>76.7</td><td>94.5</td><td>0.80</td></tr><tr><td>8</td><td>106.1</td><td>101.4</td><td>1.48</td><td>8</td><td>75.8</td><td>97.4</td><td>5.16</td><td>8</td><td>95.6</td><td>98.5</td><td>0.82</td></tr><tr><td>total</td><td>83.8 ± 7.2</td><td>97.9 ± 2.2</td><td>88 ± 22</td><td>total</td><td>77.7 ± 7.2</td><td>96.4 ± 2.3</td><td>1.9 ± 1.5</td><td>total</td><td>89.1 ± 9.1</td><td>97.3 ± 2.4</td><td>1.03 ± 0.85</td></tr></table>

In total, it shows the errors of AI patterns to the manually unwound patterns. A value of the distance from the HU moments of 0.0 shows an identical pattern shape. It is noticeable that the wider the layer of the pattern, the more accurate the pattern generated by the AI.

# 6 Conclusion

The proposed work offers significant advances in garment manufacturing and personalization. Integrating AI-driven pattern generation from 3D scans reduces the time to construct basic garments that fit a person's exact body shape. Our tool makes many steps in the fashion design more accessible, enabling 3D artists of all experience levels to effortlessly create perfectly fitting garments for any layer. AI can shorten the development time for creating a basic pattern for a new model, but it requires a high level of experience and proper use, as well as the next step of physically producing the models for further verification

This work simplifies the garment design process and allows designers and manufacturers easy access to customized clothing production. The tool automates the translation of 3D body scans into sewing patterns. This contribution streamlines the workflow from a conception to production and opens new possibilities for personalized basic patterns at scale, making bespoke garments more accessible to a broader audience. However, the tool has limitations: it is currently only capable of generating basic patterns for garment types pre-trained into the system, which restricts the scope of design creativity. While the process boasts remarkable speed, this reliance on predefined models limits the potential for truly unique garment creation. Additionally, the import process for 3D scanned individuals is not fully automated, introducing a manual step that can slow the workflow and potentially introduce errors. The integration of AI technology into pattern construction is a significant step towards making the fashion design more inclusive and accessible. In further research, pattern construction of pants, seam placement options, and sleeves will also be included.

In conclusion, developing and implementing the Al-supported tool for transforming 3D scans into customized patterns represents a significant advance in the digital clothing technology. The experiments conducted have shown that the tool is capable of generating precise and accurate patterns that are comparable to manually processed patterns. The quantitative analysis of the perimeter, area and HU moment provides evidence-based results that confirm the tools reliability and effectiveness. Knowledge gained from these experiments provides a solid basis for further optimization and adaptation of the tool to further improve its performance and applicability in practice. Overall, this research project contributes to the advancement of digital transformation in the apparel industry and paves the way for future innovations in the digital apparel technology.

In addition to verifying the patterns by physically producing them, the aim of further investigations is also to examine other patterns, such as multilayer garments and design model patterns. The current tool is only used to generate basic patterns.

# Author Contributions

M. Danner – scripting, simulation, writing– review and editing; E. Brake – construction, resources, draft preparation, writing – review and editing; G. Kosel – shell surface generation, data curation, writing – review and editing; Y. Kyosev, K. Rose, M. Ratsch and H. Cebulla: supervision, review and editing, project administration. All authors have read and agreed to the published version of the manuscript.

# Conflicts of Interest

The authors declare no conflict of interest.

# References

1. Speicher-Utsch, S. Kosten sparen mit 3D-Design. TextilWirtschaft 2017. Available online: https://www.textilwirtschaft.de/business/news/Technologie-3D-Produktentwicklungnimmt-Fahrt-auf-205495   
2. Moltenbrey, F.; Tilebein, M. Potenziale und Herausforderungen neuer digitaler Interaktionssysteme im Kollektionsentwicklungskrozess der Bekleidungsindustrie. *Mensch-Technik-Interaktion in der digitalisierten Arbeitswelt*, Schriftenreihe der Wissenschaftlichen Gesellschaft für Arbeits- und Betriebsorganisation (WGAB) e.V., Freitag, M. (Ed.), GITO mbH Verlag Berlin 2020, pp. 59-78.   
3. Hofenbitzer, G. Grundschnittte und Modellentwicklungen. Verlag Europa-Lehrmittel, Haan-Gruppen 2018.

4. Huang, H.Q.; Mok, P. Y.; Kwok, Y. L.; Au, J. S. Block pattern generation: From parameterizing human bodies to fit feature-aligned and flattenable 3D garments. Computers in Industry 2012, 63(7), 680-691.   
5. Choi, Y. L.; Nam, Y.; Choi, K. M.; Cui, M. H. A method for garment pattern generation by flattening 3D body scan data. In Digital Human Modeling: First International Conference on Digital Human Modeling, ICDHM 2007, Held as Part of HCI International 2007, Beijing, China, July 22-27, 2007. Proceedings 1, pp. 803-812. Springer Berlin Heidelberg, 2007.   
6. Mosinki, E.; Pohl, H. Grundlagen der Bekleidungskonstruktion: System Optikon. Institut für Textil- und Bekleidungswesen 1992.   
7. Kosel, G.; Rose, K. Development and Usage of 3D-Modeled Body Shapes for 3D-Pattern Making. Proc. of 3DBODY.TECH 2020 - 11th Int. Conf. and Exh. on 3D Body Scanning and Processing Technologies, Online/Virtual, 17-18 Nov. 2020, #14. DOI: https://doi.org/10.15221/20.14.   
8. Burgo, F. Il figurino di moda. Studio delle proportioni, tecniche di colorazione. Donna, uomo, bambino/a, accessori. Ediz. italiana e inglese, ISBN 8890010169 Instituto di Moda Burgo; Bilingual Edition, 2005.   
9. Narain, R.; Samii, A.; O'Brien, J. F. Adaptive Anisotropic Remeshing for Cloth Simulation. ACM Transactions on Graphics 2012, 31(6), 152.   
10. Wang, H. M.; Ramamoorthi, R.; O'Brien, J. F. Data-Driven Elastic Models for Cloth: Modeling and Measurement. ACM Transactions on Graphics 2011, 30(4), 71.   
11. Liu, L. J.; Xu, X. Y.; Lin, Z. J.; Liang, J. B.; Yan, S. C. Towards Garment Sewing Pattern Reconstruction from a Single Image. ACM Transactions on Graphics 2023, 42, 200.   
12. Korosteleva, M.; Lee, S.-H. Dataset of 3D Garments with Sewing Patterns (1.0). Zenodo 2021. DOI: https://doi.org/10.5281/zenodo.5267549.   
13. Korosteleva, M., Lee, S. H. NeuralTailor: Reconstructing sewing pattern structures from 3D point clouds of garments. ACM Transactions on Graphics 2022, 41(4), 158.   
14. Donner, E. Handbuch für die Bekleidungsindustrie und den Bekleidungs-Einzelhandel: Ein Lehr- und Nachschlagewerk für die gesamte Herren- und Knabenbekleidung. Springer 2013.   
15. Ebner Media Group GmbH & Co. KG. https://www.muellerundsohn.com/ (accessed 2024-30-03)   
16. Hohenstein Laboratories GmbH & Co. KG. https://www.hohenstein.de/de/kompetenz/passform/schnitt-service (accessed 2024-30-03)   
17. Zhang, F. F. Designing in 3D and Flattening to 2D Patterns. PhD thesis, North Carolina State University, 2015.   
18. Optitex, 2024. 3D PRODUCT CREATION SUITE. https://optitex.com/solutions/odev/3d-production-suite/ (accessed 2024-30-03)   
19. Assyst GmbH. https://www.assyst.de/de/produkte/cad/index.html (accessed 2024-30-03)   
20. LECTRA. https://www.lectra.com/en (accessed 2024-30-03)   
21. Browzwear Solutions Pte Ltd. https://browzwear.com/ (accessed 2024-30-03)   
22. CLO Virtual Fashion. https://www.clo3d.com/en/ (accessed 2024-30-03)   
23. Blender https://www.blender.org/ (accessed 2024-30-03)   
24. Abdel Kader, E. A. S.; Mohamed, R. H.; Ali, R. The Role of Artificial Intelligence (Al) Applications in fashion design and Forecasting in the garment industry, An Analytical study. International Design Journal 2022, 12, 203-214.