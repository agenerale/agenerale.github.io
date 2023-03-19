---
layout: page
title: Data Fusion
description: Inference of visco-elastoplastic multimodal damage model parameters with heterogeneous experimental data.
img: assets/img/afrl_data_fusion/figure4.png
importance: 2
category: work
scholar:
  bibliography_template: {{reference}}
---

Most materials experimental datasets tend to be very sparse and a fragmented collection of disparate measurements taken over a number of years, just owing to the prohibitive cost of performing a number of lengthy tests on large samples. Of course, this is something high-throughout experimental techniques are attempting to address {% cite rossi_study_2019 --file cited_posts %}, but we'll leave that discussion to another time.

This work dealt with a pretty standard experimental dataset consisting of 7 tests performed with simultaneous tension-torsion loading of an Oxide/Oxide ceramic matrix composite (CMC) (Images below of microstructure and sample post-test). Further details regarding testing, and images displaying the test set up and microstructure considered can be found in the thesis of DeRienzo {% cite derienzo_bi-axial_2013 --file cited_posts %}.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/afrl_data_fusion/nextelmicro.png" title="micro-CT image of Oxide/Oxide microstructure" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/afrl_data_fusion/derienzotesting.png" title="Tubular specimens with strain gauge placement." class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Importantly, all of the test were performed with distinct loading rates, such that there were no replicates of any singular test. The complete test parameters can be seen below, showing the axial and torsional loading rates:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/afrl_data_fusion/table1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    An overview of the workflow to generate stochastic virtual statistical volume elements (SVEs) (the concept of an SVE is described in further depth in {% cite tallman_14_2020 --file cited_posts %}) for the 5-harness satin composite studied.
</div>

The question being asked, was then, how can we then best identify the set of constitutive model parameters $$\boldsymbol{\theta}$$ that would best explain the 7 experimental observations?

This sort of questions is the prototypical question broadly classified as an **inverse problem**. As you might imagine, this type of problem shows up *all* the time in a broad array of scientific fields - really when we're looking to model just about anything.

In it's simplest form, this type of problem looks like:

$$ y = \mathcal{H}(\boldsymbol{\theta}) $$

where $$y$$ is the observed output, $$\mathcal{H}$$ the forward model (our damage model), and $$\boldsymbol{\theta}$$ the free parameters of our forward model we're seeking to identify. Posed even in this deceptively simple form, this problem is ill-posed, such that multiple combinations of $$\boldsymbol{\theta}$$ may lead to *equivalent* observed outputs $$y$$. 



**this project page is still under construction - complete details of this work can be found in {% cite generale_bayesian_2022 --file cited_posts %}**

References
----------

{% bibliography --file cited_posts --cited_in_order %}
