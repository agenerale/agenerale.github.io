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
    Summary of testing configurations for the available dataset.
</div>

The question being asked, was then, how can we then best identify the set of constitutive model parameters $$\boldsymbol{\theta}$$ that would best explain the 7 experimental observations?

This sort of questions is the prototypical question broadly classified as an *inverse problem*. As you might imagine, this type of problem shows up *all* the time in a broad array of scientific fields - anytime we have some governing mathematical model for which we're seeking to identify the parameterization best explaining experimental measurements.

In it's simplest form, this type of problem looks like:

$$ \boldsymbol{\theta} = \mathcal{H}^{-1}(y) $$

where $$y$$ is the observed output, $$\mathcal{H}$$ the forward model (our damage model), and $$\boldsymbol{\theta}$$ the free parameters of our forward model we're seeking to identify. Posed even in this deceptively simple form, this problem is ill-posed, such that multiple combinations of $$\boldsymbol{\theta}$$ may lead to *equivalent* observed outputs $$y$$. Unfortunately, we often also do not observed a pure output signal, but rather some corrupted version of $$y$$, only adding additional complexity. 

Fortunately, Bayesian inference provides a complete statistical inferential methodology for addressing such inverse problems in a probablistic sense, where the set of model parameters are treated as stochastic variables {% cite kennedy_bayesian_2001 stuart_inverse_2010 arridge_solving_2019 --file cited_posts %}, enabling us to identify inverse solutions while accounting for multiple sources of uncertainty.

Unfortunately, the current experimental dataset without replicates required that the experimental uncertainty also be modeled, and their variances similarly treated as stochastic variables. In order to account for the variability between samples tested, a mixed effects model of the form

$$ s^E_{ij} = \beta_i s^M_{ij}(\boldsymbol{\theta}) + \xi_{ij} $$

where $$i$$ indexes each experimental sample/test, $$j$$ points along the discretized stress-strain response, $$s^E$$ the experimentally observed stresses, $$s^M$$ the model output stresses, $$\xi_{ij} \sim \mathcal{N}(0,\sigma_{\xi}^2)$$ a corrupting noise, and $$\beta_{i} \sim \mathcal{N}(0,\sigma_{\beta}^2)$$ a multiplicative factor perturbing the stress-strain trajecoty based on intra-sample variability. Due to the normality assumption, the random effects and stochastic error can be integrated out from the model, leaving a multivariate Gaussian formulation for the likelihood as

$$ p(\boldsymbol{s}^E \vert \boldsymbol{\sigma},\sigma_{\beta}^2,\sigma_{\xi}^2) = \frac{1}{(2\pi)^{2LK}\sqrt{\vert \boldsymbol{\Sigma} \vert}} \exp{((\boldsymbol{s}^E - \boldsymbol{s}^M(\boldsymbol{\theta}))^\top \boldsymbol{\Sigma}^{-1} (\boldsymbol{s}^E - \boldsymbol{s}^M(\boldsymbol{\theta})))}$$

with $$L$$ and $$K$$ being the number of points used in the discretization of the experimental curves, and the number of experimental runs to be evaluated, respectively. The structure of the covariance matrix could then be derived as

$$ \boldsymbol{\Sigma} = \mathbb{E} [\boldsymbol{s}^E {\boldsymbol{s}^E}^\top] - \mathbb{E} [\boldsymbol{s}^E] {[\boldsymbol{s}^E]}^\top = \boldsymbol{s}^M {\boldsymbol{s}^M}^\top \odot (\sigma_{\beta}^2 \boldsymbol{I}_K \otimes \boldsymbol{J}_{2L}) + \sigma_{\xi}^2\boldsymbol{I}_{2LK} $$

and finally, inference can be performed according to

$$ p(\boldsymbol{\sigma},\sigma_{\beta}^2,\sigma_{\xi}^2 \vert \boldsymbol{s}^E) \propto p(\boldsymbol{s}^E \vert \boldsymbol{\sigma},\sigma_{\beta}^2,\sigma_{\xi}^2) p(\boldsymbol{\theta}) p(\sigma_{\beta}^2,\sigma_{\xi}^2) $$

The posterior was then approximated by a parallel chain Markov Chain Monte Carlo (MCMC) algorithm with an *affine invariant* sampler {% cite foreman-mackey_emcee_2013 --file cited_posts %}. The resulting estimated high-dimensional posterior distribution can be visualized through a scatter plot matrix, where the *Maximum a Posteri* (MAP) value is highlighted in blue.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/afrl_data_fusion/figure3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Posterior distribution identified through MCMC.
</div>

Finally, samples of this posterior were then pushed-forward through the constitutive model to evaluate the ability of the statistical calibration to simultaneously explain the multiple data experimental sources.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/afrl_data_fusion/figure4.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Push-forward of posterior distribution through forward constitutive model displaying simultaneous coverage of all experimental observations.
</div>

A more complete discussion of this work can be found in {% cite generale_bayesian_2022 --file cited_posts %}.

References
----------

{% bibliography --file cited_posts --cited_in_order %}
