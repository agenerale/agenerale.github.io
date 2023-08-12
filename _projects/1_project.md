---
layout: page
title: Thermal Conductivity
description: Using active learning to develop structure-property surrogate model linkages
img: assets/img/conductivity/figure1.jpg
importance: 1
category: work
scholar:
  bibliography_template: {{reference}}
---

One of the first projects of my Ph.D. entailed developing several Gaussian process ($$\mathcal{GP}$$) surrogate models in order to predict the effective thermal conductivity of stochastic virtually generated chemical vapor infiltration (CVI) densified SiC/SiC ceramic matrix composite (CMC) microstructures. This post should hopefully serve as an easy to digest version of the paper *"surrogate models for microstructure-sensitive effective thermal conductivity of woven ceramic matrix composites with residual porosity*" {% cite generale_surrogate_2021 --file cited_posts %}, but of course the article provides a more thorough treatment of the work.

CMCs and especially CVI-densified SiC/SiC CMCs are incredibly fascinating as it's one of the few available structural material systems which can operate in high-temperature oxidizing environments for prolonged periods. As with all material systems, there are of course a few trade offs:
1. Exceedingly long densification times required in the reactor.
2. Residual porosity often of the order of 8-12% {% cite morscher_advanced_2007 bansal_ceramic_2014 --file cited_posts %}.

The end result is that it's particularly challenging to have good through-thickness (or out-of-plane) thermal conductivity with such a high percentage of residual porosity. This porosity is also inherently coupled to the weave architecture, or the resulting spatial arrangement of constituents in the microstructure. Of course for many high heat-flux applications, we'd be looking to maximize thermal conductivity out-of-plane ($$k_{33}$$) so that we can adequately cool the surface temperature. The natural next question to ask might be:

How can we quickly search through various spatial arrangements of constituents to accomplish this goal (i.e. maximize $$k_{33}$$)?

We could either:
- Numerically solve governing PDE (ex. Finite-Element Method) for each and every possible microstructure (I should mention, more refined methods exist to search this space, such as Topology Optimization (TO), although TO presents its own issues in terms of cell-discretization of the input domain, initialization, convergence, and is still computationally demanding) {% cite zhu_two-scale_2017 --file cited_posts %}.
- Develop analytical expression relating pre-defined features in the microstructure to target property, such as tow width, height, etc., alongside empirical calibration factors.
- Develop low-cost physics-based machine learning model relating microstructure features to the desired property.

This work opts to develop this mapping from structure to property, $$G_\theta: \mathcal{S} \rightarrow \mathcal{P}$$ along the lines of the latter option, parameterized by $$\theta$$. 

The first step requires necessarily requires data curation/generation. This was done using the open-source textile generator TexGen {% cite lin_modelling_2011 --file cited_posts %} along with certain downstream post-processing steps to impart semi-realistic residual porosity. The generating process looked similar to:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An overview of the workflow to generate stochastic virtual statistical volume elements (SVEs) (the concept of an SVE is described in further depth in {% cite tallman_14_2020 --file cited_posts %}) for the 5-harness satin composite studied.
</div>

All told, 3125 SVEs were generated with an initial Latin Hypercube Design on the generating parameters, tow width, height, spacing, ply compaction, and matrix deposition thickness to cover varied potential microstructures.

Low-dimensional microstructure descriptors were identified through a statistical representation of spatial correlations within each microstructure (more formally, 2-point spatial correlations) {% cite torquato_random_2002 adams_microstructure-sensitive_2013 kalidindi_materials_2015 --file cited_posts %}. Unfortunately, while 2-point spatial correlations (or 2-point statistics) are incredibly descriptive, above and beyond certain microstructural features, they often tend to <i>increase</i> or maintain the dimensionality of the discretized microstructure. To provide features more amenable as inputs to a ML model, Principal Component Analysis (PCA) is then performed on the set of 2-point spatial correlations.

Each the collection of SVEs can then be transformed to a point cloud in PC-space, separating microstructures in an ordered basis by the explained variance in the 2-point spatial correlations, which can be visualized as:

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/conductivity/figure4.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Microstructure ensemble in PC-space, demonstrating ability of identified basis to separate microstructures in an ordered well-behaved space.
</div> 

The mapping $$G_\theta$$ was defined to be 3 independent $$\mathcal{GP}$$s, each predicting
 one component of the orthotropic thermal conductivity of the SVE, with $$\theta$$ representing
 the set of hyperparameters of the automatic relevance determination squared exponential (ARD-SE) kernel {% cite bishop_pattern_2006 --file cited_posts %}.
 The computational cost involved in model building was minimized through sequentially identifying an
 optimal experimental design in the input space through active learning. This strategy minimizes the
 number of expensive black-box function evaluations, or the number of FE-simulations required, through
 intelligently selecting locations in the input space for which the model is the most uncertain.
 
In practice, this maximum posterior uncertainty strategy looks like:
1. Train the $$\mathcal{GP}$$s on a small initialization dataset (~10$$d$$ where $$d$$ is the input feature dimension).
2. Perform predictions with the current model, $$\mathcal{GP}$$s, on the remaining potential microstructures.
3. Identify the microstructure $$\boldsymbol{\alpha}^*$$ with the largest posterior uncertainty, and evaluate the black-box FE-simulation for that microstructure.

Running through this iterative process can save redundant FE function evaluations and computing hours
 running microstructures which are not informative of the underlying structure-property linkage.
 As a demonstration, below are plots showing rapid convergence of error metrics and $$\theta$$
 in the first ~200 total microstructures - a significant reduction over the complete available dataset.

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



References
----------

{% bibliography --file cited_posts --cited_in_order %}
