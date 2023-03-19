---
layout: page
title: Thermal Conductivity
description: Using active learning to develop structure-property reduced-order model linkages
img: assets/img/conductivity/figure1.jpg
importance: 1
category: work
---

One of the first projects during my Ph.D. entailed developing several Gaussian process ($$\mathcal{GP}$$) reduced-order models in order to predict the effective thermal conductivity of stochastic virtually generated chemical vapour infiltration (CVI) densified SiC/SiC ceramic matrix composite (CMC) microstructures. This post should hopefully serve as an easy to digest version of the paper <a href="https://agenerale.github.io/assets/pdf/2021_Generale_Reduced-order_models_for_microstructure-sensitive_thermal_conductivity_with_residual_porosity_CMCs.pdf">"<i>Reduced-order models for microstructure-sensitive effective thermal conductivity of woven ceramic matrix composites with residual porosity</i>"</a>, but of course the article provides a more thorough treatment of the work.

CMCs and especially CVI-densified SiC/SiC CMCs are incredibly fascinating as it's one of the few available structural material systems which can operate in high-temperature oxidizing environments for prolonged periods. As with all material systems, there are of course a few trade offs:
1. Exceedingly long densification times required in the reactor.
2. Residual porosity often of the order of 8-12%.

The end result is that it's particularly challenging to have good through-thickness (or out-of-plane) thermal conductivity with such a high percentage of residual porosity. This porosity is also inherently coupled to the weave architecture, or the resulting spatial arrangement of constituents in the microstructure. Of course for many high heat-flux applications, we'd be looking to maximize thermal conductivity out-of-plane ($$k_{33}$$) so that we can adequately cool the surface tamperature. The natural next question to ask might be:

How can we quickly search through various spatial arrangements of constituents to minimize $$k_{33}$$?

We could either:
- Numerically solve governing PDE (ex. Finite-Element Method) for each and every possible microstructure (I should mention, more refined methods exist to search this space, such as Topology Optimization (TO), although TO presents its own issues in terms of cell-discretization of the input domain, initialization, convergence, and is still computationally demanding).
- Develop analytical expression relating pre-defined features in the microstructure to target property, such as tow width, height, etc., alongside empirical calibration factors.
- Develop low-cost physics-based machine learning model relating microstructure features to the desired property.

This work opts to develop this mapping from structure to property, $$G_\theta: \mathcal{S} \rightarrow \mathcal{P}$$ along the lines of the last option, parameterized by $$\theta$$. 

Before I get to discussing the microstructure descriptors selected, of course, as in any ML problem, the first step requires data curation/generation. This was done using the open source textile generator TexGen along with certain downstream post-processing steps to impart semi-realistic residual porosity. The generating process looked a little like:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An overview of the workflow to generate stochastic virtual statistical volume elements (SVEs) for the 5-harness satin composite studied.
</div>

All told, 3125 SVEs were generated with an initial Latin Hypercube Design on tow width, height, spacing, ply compaction, and matrix deposition thickness to cover varied potential microstructures.

Low-dimensional microstructure descriptors were identified through a statistical representation of spatial correlations within each microstructure (more formally, 2-point spatial correlations). Unfortunately, while 2-point spatial correlations (or 2-point statistics) are increadibly descriptive, above and beyond certain microstructural features, they often tend to <i>increase</i> or maintain the dimensionality of the diescretized microstructure. To provide features more amenable as inputs to a ML model, Principal Component Analysis (PCA) is then performed on the set of 2-point spatial correlations.

Each the collection of SVEs can then be transformed to a point cloud in PC-space, seperating microstructures in an ordered basis by the explained variance in the 2-point spatial correlations, which can be visualized as:

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure4.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Microstructure ensemble in PC-space, demonstrating ability of identified basis to separate microstructures in an ordered well-behaved space.
</div> 

The mapping $$G_\theta$$ was defined to be 3 independent $$\mathcal{GP}$$s, each predicting one component of the orthotropic thermal conductivity of the SVE, with $$\theta$$ representing the set of hyperparameters of the ARD-SE kernel. The cost of generating training data (through FE-simulations) was minimized by pursuing an active learning sequential selection methodology, where the goal is to only run expensive FE-simulations on the microstructures *absolutely required* to define $$G_\theta$$.

In practice, this looks like:
1. Train the $$\mathcal{GP}$$s on a small intialization dataset (~5 microstructures).
2. Make predictions with the trained $$\mathcal{GP}$$s on the remaining potential microstructures.
3. Identify the microstructure $$\boldsymbol{\alpha}^*$$ with the largest posterior ($$p(\boldsymbol{k} \vert \boldsymbol{\alpha}_{1:3})$$) uncertainty, $$\boldsymbol{\alpha}^{*} = \textrm{argmax var}(\boldsymbol{k} \vert \boldsymbol{\alpha}_{1:3})$$, and run a FE-simulation for that microstructure.

Running through this iterative process can save lots of unnecessarily computing hours running microstructures which aren't going to aird in training the underlying models! Just to demonstrate, here are some plots showing rapid convergence of error metrics and $$\theta$$ in the first ~200 total microstructures - a massive reduction over the complete available dataset.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure7.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Rapid convergence of ARD-SE hyperparameters model error metrics with sequential selection of training microstructures by maximum posterior variance.
</div> 

Of course, the last step was to benchmark this trained model against available analytical formulations for effective thermal conductivity of CMCs (H2L). 

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Parity plots of thermal conductivity prediction for each of the trained models.
</div> 
