---
layout: page
title: Inverse Stochastic Microstructure Design
description: Stochastic design of materials through deep variational inference with hierarchical machine-learned latent spaces.
img: assets/img/inv_design/framework_pictoral.png
importance: 1
category: work
scholar:
  bibliography_template: {{reference}}
---

Inverse Microstructure Design problems are ubiquitous in materials science; for example, property-driven microstructure design requires the inversion of a structure--property (SP) linkage (i.e., a mapping from microstructure to its homogenized property). However, prior frameworks have struggled to address this problem's unique combination of challenges: the high dimensionality and stochasticity of microstructures, nonuniformly-distributed initial datasets, and ill-conditioning of the inversion. This work proposes a computational framework for addressing Inverse Microstructure Design problems using a Bayesian methodology. This framework is comprised of three modular components, enabling flexible extension and re-use for future work. First, a low-dimensional, informative microstructure prior is constructed by integrating domain knowledge (i.e., statistical continuum mechanics) into a distributional learning scheme. This scheme includes multiple latent representations which address the challenges inherent to representing microstructures. Second, a property-specific likelihood is defined through a Multi-Output Gaussian Process Regression surrogate model. Lastly, the conditional posterior density is efficiently learned for a given target property, and samples are generated using deep variational inference. 

As a means of demonstration, this proposed approach was applied to identifying woven ceramic matrix composites conditioned upon a set of target anisotropic thermal conductivity. This challenging example allows us to interrogate the integral role of each framework component in the overarching inversion.

**Framework**

In this section, we'll first develop the computational scaffolding necessary to treat the Stochastic Microstructure Design problem through Bayesian inference. The figure below visually summarizes the overarching strategy. For this problem, we define variable names to correspond to the task of microstructure design, where inputs of the forward model are our microstructure representation $$\boldsymbol{\alpha}$$ (i.e., principal component (PC) scores of spatial correlations), and outputs are the target effective property set $$\boldsymbol{k}$$.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/framework_pictoral.PNG" title="Depiction detailing the overall framework." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Depiction detailing the overall framework. First, a generalized microstructure prior is derived to represent a target microstructure class. A surrogate likelihood represents the salient SP linkage for this class. The combination of the microstructure prior, the likelihood, and a user-supplied target property value produces a property conditioned posterior distribution -- the solution to the inverse problem. Note that the microstructure diversity in the posterior is far smaller than the prior. The distributions shown in this figure are directly extracted from the first case study in this work.
</div> 

The proposed framework combines the respective strengths of several deep generative model architectures to be interpretable, modular, and robust. It is compromised of 3 main components each addressing a salient element of the Bayes' Rule computation described as

$$ p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*) = \frac{p(\boldsymbol{k}^* \vert \boldsymbol{\alpha})p(\boldsymbol{k}^*)}{\int p(\boldsymbol{k}^* \vert \boldsymbol{\alpha})p(\boldsymbol{\alpha})d\boldsymbol{\alpha}} $$

*Components*

1. A forward model $$\mathcal{H}:\boldsymbol{\alpha} \rightarrow \boldsymbol{k}$$. This is used to approximate the likelihood term, $$p(\boldsymbol{k} \vert \boldsymbol{\alpha})$$, mapping a microstructure representation, $$\boldsymbol{\alpha}$$, to effective properties, $$\boldsymbol{k}$$.
2. A $$\beta$$-VAE based microstructure prior, $$p(\boldsymbol{\alpha})$$. This crucial mechanism derives a usable prior distribution over $$\boldsymbol{\alpha}$$ that regularizes the inverse problem and provides data compression and efficient sampling.
3. Finally, a flow-based generative model trained on a specific target property $$\boldsymbol{k}^*$$ which, when combined with the $$\beta$$-VAE's decoder, estimates the posterior distribution $$p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*)$$.

It should be emphasized that the first two components only incur a one time upfront training cost (e.g., over all possible $$\boldsymbol{\alpha},  \boldsymbol{k}$$), while the inference of the posterior $$p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*)$$ (the third component) must be repeated for each target property $$\boldsymbol{k}^*$$. An amortized approach {% cite radev_bayesflow_2022 --file cited_posts %} would certainly be possible, but is outside the scope of this work.

In this work, we adopt the MKS framework for Materials Informatics {% cite kalidindi_materials_2015 adams_microstructure-sensitive_2013 --file cited_posts %} and quantify material microstructures using their 2-point spatial correlations. To simplify downstream computations, these statistics were then compressed into a primary lower-dimensional design manifold using PCA {% cite khosravani_development_2017 latypov_data-driven_2017 paulson_reduced-order_2017 iskakov_application_2018 --file cited_posts%}. This first latent space compactly represents microstructures, and, more specifically, their 2-point spatial correlations, via their principal components, $$\boldsymbol{\alpha}$$. The ultimate goal of the presented framework is to recover the distribution of 2-point spatial correlations corresponding to a desired property set. The usage of correlations removes the notion of an origin and provides a translation-invariant representation of microstructures (which is desirable for predicting global properties). The PCA process removes the notion of space at all, exchanging high-dimensional spatial fields for dense coefficients corresponding to an orthonormal set of spatial basis functions. This provides a low-rank representation of microstructure which is interpretable, analytically well-founded, and extremely cheap to compute.

However, a much larger PC representation was adopted than usual: in the presented case studies $$\sim 1000$$ PC scores were used instead of the $$\sim 10$$ in most previous MKS efforts {% cite gupta_structureproperty_2015 latypov_materials_2019 yabansu_digital_2020 marshall_autonomous_2021 paulson_reduced-order_2017 generale_reduced-order_2021 kalidindi_feature_2020 --file cited_posts %}. This switch reflects the difference in goals between this and previous efforts. These remaining principal components define finer microstructure details; while they are unnecessary for building forward models because such details have little effect on resulting properties, they are often necessary for downstream tasks, such as generation of microstructure instances {% cite robertson_efficient_2022 robertson_localglobal_2023 harrington_application_2022 --file cited_posts %} corresponding to the inverse solution identified by the proposed framework. Therefore, retaining extra PC scores is necessary to produce a useful solution to the inverse problem. This is still a significant compression when compared to the $$101^3$$ voxels of the original microstructures.

**A Surrogate Likelihood**

It is unavoidably necessary to evaluate Bayes' Rule an extremely large number of times in most practical implementations of Bayesian inference (in the absence of model conjugacy.). This requirement poses a significant challenge as the structure-to-property mapping, needed in the likelihood computation, is often an expensive, high-fidelity numerical simulation {% cite rossin_bayesian_2021 --file cited_posts %}. Fortunately, the construction of reduced-order surrogate models alleviates this computational bottleneck. 

In this work, we use Sparse Variational Multi-Output Gaussian Process Regression (SV-MOGP) with a Spectral Mixture (SM) kernel to approximate the forward model, $$\mathcal{H}$$, from our principal component microstructure representation to property. These GP surrogates not only provide probabilistic predictions of the output, but also inform the computation of the likelihood. The expressive SM kernel was selected in this work due to its previously observed stability during extrapolation {% cite wilson_gaussian_2013 --file cited_posts %}. Such stability is often important because existing microstructure data rarely covers the design space uniformly and is often clustered into subregions {% cite paulson_reduced-order_2017 --file cited_posts %}. In the present case, this stability was critical to reaching beyond the existing microstructure clusters and facilitating the ultimate goal of materials discovery: identifying valuable, novel microstructures. 

The surrogate modelling framework has several parameters that must be tuned to facilitate successful performance. Briefly, the parameter with the most theoretical impact to the inverse problem is the number of utilized principal components. In practice, extensive prior efforts have demonstrated that only the first few principal components are necessary to successfully train surrogate forward models in the MKS framework {%cite yabansu_extraction_2017 generale_reduced-order_2021 generale_uncertainty_2023 gupta_structureproperty_2015 latypov_materials_2019  paulson_reduced-order_2017 --file cited_posts %}. With respect to the training process, this is quite useful because it means that lightweight models trained on small datasets are usually sufficient. However, this also highlights the primary source of complexity surrounding microstructure inverse problems: the problem is highly under-constrained. Through the likelihood, a given property can only inform the value of the first few principal components. For example, in the case study presented in this summary, the MOGP is fed only the first 16 PC scores; the remaining principal components must be identified by the prior. As we will discuss, this is the extremely important responsibility of the prior. 

**A Microstructure Prior**

A well-crafted prior is an essential component to the inversion process. While this is generally important for any Bayesian inference framework, it is especially important in our case because the likelihood can only inform the PC scores it takes as input. As a result, the microstructure prior plays the triple role of informing the remaining PC scores by identifying nonlinear interdependencies between principal components, encoding prior domain knowledge or constraints, and efficiently sampling the high-dimensional microstructure space.

Unfortunately, the complex topology and high dimensionality of a large principal component space makes it complicated to estimate a usable prior. As will be shown later, the principal component space generally contains multiple clusters and large cavities (regions where microstructures have not yet been instantiated). To overcome this, we introduce a second latent space; we utilize a $$\beta$$-VAE as an additional nonlinear embedding of the 2-point spatial correlations subsequent to PCA. By design this latent space affords a natural prior: the unit-Gaussian $$p(\boldsymbol{z}) = \mathcal{N}(\boldsymbol{0},\boldsymbol{I})$$. Therefore, we perform Bayesian inference in the $$\beta$$-VAE's latent space. 

The $$\beta$$-VAE architecture was selected because the $$\beta$$ hyperparameter provides a means of trading consistency of the unit-Gaussian latent structure against reconstruction performance of the encoding/decoding. This directly provides a means of tightening the prior to adjust performance of the inversion. We trained the $$\beta$$-VAE to further compress the primary PCA latent space instead of taking the full 2-point correlations as inputs because it significantly simplified the compression that the autoencoder must learn. In practice, we observed that this combined process allowed us to achieve superior performance with much smaller models. In essence, PCA acts as an orthogonal linear transformation, the first layer in a larger augmented-VAE mapping 2-point statistics to $$\boldsymbol{z}$$ and back.

Finally, we retained the usage of two latent spaces in the framework -- the primary PC space and the secondary $$\beta$$-VAE space -- instead of transitioning to just the single $$\beta$$-VAE space where we perform Bayesian inference because we found that constructing linkages from the $$\beta$$-VAE's latent space resulted in significantly degraded performance in comparison to construction solely based on the latent PC space. For the SV-MOGP model utilized in the presented case studies, training on the $$\beta$$-VAE latent space resulted in worsened mean predictions alongside higher marginal posterior uncertainty. Therefore, the likelihood $$p(\boldsymbol{k}^* \vert \boldsymbol{z})$$ used in Bayesian inference is then defined through the use of both the decoder and the SV-MOGP forward model (i.e., $$p(\boldsymbol{k}^* \vert \boldsymbol{z}) = \int p(\boldsymbol{k}^* \vert \boldsymbol{\alpha}) p_{\psi}(\boldsymbol{\alpha} \vert \boldsymbol{z}) d\boldsymbol{\alpha}$$).


**Deep Variational Inference**

The two components described above combine to fully define Bayes' Rule in the $$\beta$$-VAE based latent space. 

$$ p(\boldsymbol{z} \vert \boldsymbol{k}^*) \propto \left( \int p(\boldsymbol{k}^* \vert \boldsymbol{\alpha}) p_{\psi}(\boldsymbol{\alpha} \vert \boldsymbol{z}) d\boldsymbol{\alpha} \right) p(\boldsymbol{z}) $$

The likelihood, given by the integral above, directly incorporates the forward SV-MOGP alongside the decoder of the $$\beta$$-VAE. The prior is the unit-Gaussian, $$p(\boldsymbol{z}) = \mathcal{N}(\boldsymbol{0},\boldsymbol{I})$$ -- the heavily imposed prior distribution on the $$\beta$$-VAE's latent space. Finally, deep variational inference is performed to solve the microstructure inverse problem using a flow-based generative model to estimate the target property conditioned posterior, $$q_{\theta^*}(\boldsymbol{z}) \approx p(\boldsymbol{z} \vert \boldsymbol{k}^*)$$. The model is trained for a specific target property value, $$\boldsymbol{k}^*$$, with associated learned weights $$\theta^*$$. After training, we can generate individual solutions by sampling our approximate posterior and, subsequently, transforming them into 2-point spatial correlations by applying the $$\beta$$-VAE decoder and then the inverse PCA transformation. 

$$ p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*) \approx \int p_{\psi}(\boldsymbol{\alpha} \vert \boldsymbol{z}) q_{\theta^*}(\boldsymbol{z} \vert \boldsymbol{k}^*) d\boldsymbol{z} $$


**Microstructure Dataset**

The complete dataset contains 3,125 instantiations of an 8-ply stack of 5-harness satin (5HS) weave ceramic matrix composites (CMCs) from prior work {% cite generale_reduced-order_2021 --file cited_posts %}. Each 3D microstructure in this dataset is comprised of  $$109 \times 109 \times 109$$ voxels, each of which are occupied by one of three possible distinct material local states: tow, matrix, or pore. The microstructure ensemble was generated by varying the tow major and minor axes, tow spacing, ply spacing, and matrix thickness. This resulted in significant microstructural diversity and, consequently, effective orthotropic thermal conductivity. Additional details regarding the generative procedure and numerical simulation inputs can be found in prior work {% cite generale_reduced-order_2021 --file cited_posts %}.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/k_ensemble_paper.png" title="Property Closure." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Property closure of the microstructure ensemble considered in this work. Select microstructures are identified as test cases for the proposed stochastic microstructure design framework.
</div> 

The figure above visually summarizes the resulting property closure for this dataset (the projection of $$k_{22}$$ and $$k_{33}$$ was omitted due to the similarity of in-plane properties). Each microstructure is statistically represented by a subset of its 2-point spatial correlations; we computed $$\{f^{00}_r, f^{11}_r, f^{22}_r, f^{02}_r\}$$, where the material local state indexes $$\{0,1,2\}$$, correspond to tow, matrix, and pore, respectively. An exemplar set of spatial correlations (corresponding to the first test case discussed later) can be seen in the figure below. Part (b) highlights the X-Z cross-section displaying the uniformly deposited matrix surrounding individual tows, and the residual porosity from the approximated chemical vapor deposition process. Part (a) of the figure displays the base 5HS weave architecture, with the relative locations of cross-over points in the weave demarcated by arrows. The resulting spatial correlations in part (d) also prominently display this feature of the weave architecture across the set of 2-point spatial correlations, similarly demarcated by arrows. The secondary peaks highlighted in (a) and (d)  for the tow auto-correlation $$f^{00}_r$$, coincide with cross-over locations of the 5HS weave, with tertiary peaks across the domain reflecting the periodic nature of the microstructure itself. Importantly, the central peaks in the set of auto-correlations (i.e., $$\{f^{00}_r, f^{11}_r, f^{22}_r\}$$), reflect the volume fractions of each local state, and necessarily sum to one. A deeper analysis and interpretation of the spatial correlation maps for this class of microstructures can be found in our prior work {% cite generale_reduced-order_2021 --file cited_posts %}.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/cases_2pt.png" title="Exemplar 2-Point Spatial Correlations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Exemplar microstructure (corresponding to the first microstructure test case) and its spatial correlation maps. (a) 5HS weave displaying relationship between cross-over points, (b) X-Z cross-section of the exemplar microstructure, with tows depicted in gray, matrix in black, and porosity in white. (c) Set of 2-point spatial correlations associated with the microstructure in (b) with the central planes of the 3D field highlighted for clarity. (d) X-Y central plane from (c).
</div> 

As discussed, we reduced the dataset's dimensionality using PCA on the flattened spatial correlations. The full set of 2-point spatial correlations for each microstructure instantiation is represented by a feature vector of length $$5,180,116$$. When performing PCA, scaling factors were applied to the subsets of spatial correlations as $$\{f^{00}_r, 9.83f^{11}_r, 21.58f^{22}_r, 8.44f^{02}_r\}$$ such that the variance of each subset were equivalent (i.e., no particular subset dominated the transformation). Part (a) of the figure below illustrates projections of the microstructure ensemble in the first 3 PCs as well as cross-sections of the test microstructures. We kept $$1024$$ PC scores in order to overwhelmingly capture any salient variations in the dataset and preserve higher-order connections between PCs.

This figure highlights the motivation for selecting these particular test cases, which are arranged in increasingly sparse regions in PC space. Consequently, this represents increasing levels of difficulty in identifying the correct solution to the inverse problem. In particular, it can be observed that case \#1 corresponds to a dense region in the microstructure ensemble, and case \#3 is locally isolated, with case \#2 set to an intermediate location on the extremity of the target effective properties.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/pc_ensemble.png" title="Ensemble in PC Space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    PCA representation of the ensemble of microstructures along with cross-sections of the identified test-case microstructures. (a) Projected view of the synthetic microstructure ensemble in the PC0 vs. PC1 subspace, (b) projected view of the synthetic microstructure ensemble in the PC subspace spanned by PC0-2 and (c) cut-away views of the test-case microstructures, with yellow corresponding to the tows, gray to matrix, and porosity left transparent.
</div> 


**Case Study 1**

In each case study, we jointly select a target property and microstructure pair, $$\{m_1,\boldsymbol{k}^*_1\}$$ from the training dataset for the prior and likelihood. The first case study tests the framework's performance on a modestly challenging inverse problem in which a valid solution posterior (conditioned on $$\boldsymbol{k}_1$$) should be readily identifiable; moreover, this posterior should contain the known microstructure $$m_1$$. We select the pair from a densely-sampled region in both the property closure and the microstructure ensemble. The primary challenge that arises in this situation is degeneracy in the solution -- a wide number of microstructures can produce a narrow range of properties. Therefore, in addition to benchmarking the framework's performance, this situation tests its capacity to identify a wide and likely multimodal posterior. 

The posterior $$p(\boldsymbol{z} \vert \boldsymbol{k}^*_1)$$ was identified through the procedure preiously laid out, enabling us to draw conditioned samples over the latent space of the $$\beta$$-VAE. Analyzing this space is complicated because the $$\beta$$-VAE latent dimensions do not inherently possess any ranking corresponding to information (as opposed to PC scores, which are ordered by explained variance). Due to these limitations, we begin by identifying the maximally informative dimensions of the full $$64$$-dimensional posterior distribution. The dimensions of $$\boldsymbol{z}$$ exhibiting the largest deviations from unit Gaussian were identified by ranking in descending order the absolute value of their marginal higher-order moments\footnote{Dimensions whose moments match the initial Gaussian distribution are necessarily uniformative.}, summarized in the figure below. As can be seen in part (b), only four (of the 64 total) latent dimensions of $$\boldsymbol{z}$$ display strong sensitivity to this specific target property.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/1556_corner4_skew_plot.png" title="Posterior Scatter Plot VAE Latent Space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    (a) Posterior distribution in the VAE latent space for the dimensions with the highest absolute skewness values in the marginalized distributions. The posterior is identified in blue marginalized histograms on the main diagonal and 2D projections on the lower triangle, with the MAP instantiated microstructure position demarcated in red, and the initial ensemble in black. (b) Ranked decaying absolute skewness values across the 64 latent dimensions alongside corresponding Fisher kurtosis values.
</div> 

Part (a) displays projections of the posterior's most informative dimensions. Their distribution is non-Gaussian, capturing the complex interdependencies between these parameters and the property in this nonlinear design space. This is most prominently displayed in the off-diagonal projections. Importantly, the framework passes our simple initial test: the Maximum a Posteriori (MAP) location of the instantiating microstructure for this property is readily located within the posterior distribution in all dimensions -- including those not displayed. Excitingly, the posterior is able to recover more than just the test microstructure. The identified posterior is relatively wide (as expected). Given the diversity of microstructures near the selected $$m_1$$ in both the PC and property spaces of the training set, we would expect to find a wide range of spatial correlations satisfying such a target property set. The framework autonomously identifies this degeneracy and quantifies it neatly in the returned posterior. This is a primary benefit of posing the problem of inverse microstructure design as a stochastic inverse problem. By utilizing probabilistic tools, the framework can estimate uncertainty through the inversion and characterize the variability of potential microstructures matching a design target.

We next visualize posterior samples in the primary PC space (i.e., samples from $$p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*_1)$$ obtained through use of the VAE decoder $$p_{\psi}(\boldsymbol{\alpha} \vert \boldsymbol{z})$$), shown in the figure below. We reemphasize that this is our primary space of interest because of the poor topology of the $$\beta$$-VAE latent space {% cite fu_cyclical_2019 spinner_towards_2018 connor_variational_2021 --file cited_posts %}. Contrasting the distributions in the two latent spaces, we see several important benefits of the bi-level inference framework. The nonlinear decoding transformation warps the originally-unimodal posterior $$p(\boldsymbol{z} \vert \boldsymbol{k}^*_1)$$ into a complex, multimodal, disconnected distribution in PC space. Additionally, performing inference in the VAE latent space ameliorates several known limitations of the stochastic variational inference methodology, namely, zero-forcing effects of the KL-divergence {% cite zhang_advances_2019 sun_deep_2021 --file cited_posts %}, and spurious connectivity in the target density of normalizing flows {% cite papamakarios_normalizing_2021 rezende_variational_2016 sun_-deep_2022 --file cited_posts %}. The compacted nature of the space over which inference is being performed significantly reduces the chances of an exact target density displaying discontinuities or multimodal behavior. 

These disconnected components of the posterior in PC space can be directly observed in the marginal distributions, especially for $$\alpha_3$$. Importantly, in this space, the highest-probability regions of the identified posterior surround the original microstructure $$m_1$$, with increasing variability in higher-order PC terms. This is expected due to the intimate link between the highest PC scores and resulting effective property {% cite khosravani_development_2017 latypov_data-driven_2017 paulson_reduced-order_2017 iskakov_application_2018 latypov_materials_2019  yabansu_digital_2020, marshall_autonomous_2021  paulson_reduced-order_2017 generale_reduced-order_2021 kalidindi_feature_2020 castillo_bayesian_indentation_2019 --file cited_posts %}. Moreover, we observe that the posterior seems to bridge cavities in the design space, most clearly seen in the projected view $$\{\alpha_0,\alpha_1\}$$. Sets of 2-point spatial correlations falling inside these design cavities demonstrates the framework's ability to extend beyond the training set and highlight novel microstructures with similar properties to $$m_1$$. 

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/corner_pcs_double4_1556.png" title="Posterior Scatter Plot PC Space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Posterior distribution transformed through the decoder. The posterior is identified in blue margenalized histograms on the main diagonal and 2D projections on the lower triangle, with the instantiated microstructure position demarcated in red, and the initial ensemble in black.
</div> 

Next, we interrogate how well the posterior predictions match the desired target property set. To answer this we push posterior samples $$\boldsymbol{\alpha}^{(i)} \sim p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*_1)$$ through the surrogate forward model and compare the (average) predicted property of each generated sample with the target property $$\boldsymbol{k}^*_1$$. 

For comparison, we also consider microstructures from the training dataset which lie within the convex hull (along the first 6 PC dimensions) of $$\boldsymbol{\alpha}^{(i)}$$ generated from the posterior. A total of 77 structures were identified meeting this criterion. An enlarged view of the posterior distribution in the first 3 dimensions can be seen in below in part (a), with the structures for which FE results are available highlighted in black. Interestingly, the inverse solution posterior was able to identify structures both from the region of the instantiating microstructure, as well as potential structures bridging the design cavity, and highlighting additional structures with vastly different values of $$\alpha_1$$. The push-forward of the posterior distribution through the forward model can be seen in part (b), alongside the target set of orthotropic thermal conductivity. For each property, the distribution of mean predictions of the SV-MOGP encompasses the design target, and the distribution of FE-simulated results overlaps the entire posterior of the SV-MOGP. Select FE results can be seen to lie outside of the push-forward $$\pm 2\sigma$$ of the posterior for $$k_{33}$$, although we believe this is due to the limitations of this selection criterion. Due to computational overhead in identifying the convex hull {% cite noauthor_qhull_nodate --file cited_posts %} of the posterior samples across the high-dimensional space of $$\boldsymbol{\alpha}$$, it was only feasible to search in the first 6 PC-dimensions; therefore these property-outlier FE results could be caused by deviations in their higher-order PC scores. Additional evidence for this theory was noted during construction of the forward model, where improved performance was found with the inclusion of up to 16 PC scores.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/1556_posterior_property.png" title="Posterior Push-Forward Property Space" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Validation of the push-forward of the posterior distribution through the SV-MOGP. (a) Posterior distribution in blue overlaid upon the prior distribution of the microstructure enseble, the instantiating microstructure highlighted in aqua, and sampled points in the first 5 dimensions of the posterior convex hull are identified in black. (b) Histograms of the posterior of the SV-MOGP with mean prediction in blue, FE-simulated results in red, and the target property set as a dashed vertical black line.
</div> 

Finally, we inspect the solution posterior in the ultimately-desired space of 2-point spatial correlations. To do so, samples of $$p(\boldsymbol{\alpha} \vert \boldsymbol{k}^*_1)$$ are simply mapped back to 2-point correlations using the PC basis. A number of visualizations for this distribution are presented below. One-dimensional slices (along the central X axis) of sampled correlations are shown in part (a), and means and variances of 2D slices (along the central X-Y plane) can be seen in part (b). In particular, part (a) demonstrates that the posterior exhibits very little variation in the matrix autocorrelations, $$f_r^{11}$$, but is dispersed in the tow and porosity autocorrelations, $$f_r^{00}$$ and $$f_r^{22}$$. 

Parts (b) and (c) provide a more information-dense view of the identified posterior. In particular, the mean 2-point spatial correlations in (b) display a blurring effect of the secondary and tertiary peaks, most prominently observed in comparison with those of a single training microstructure. This effect can also be seen to increase in intensity with $$\lVert \boldsymbol{r} \rVert_2$$. In other words, the posterior has identified a set of microstructures with variable tow spacing and tow width which still satisfy the target property set. The variance of this central plane across the set of correlations in (c) only reinforces this conclusion. The streaking exhibited in $$\textrm{var}[f_r^{00}]$$, $$\textrm{var}[f_r^{11}]$$, and $$\textrm{var}[f_r^{22}]$$ further highlights the variability in the position of 5HS cross-over locations as a result of shifting tow spacing and tow width. These are all useful observations for engineering activities such as quality control: large trade-off between the volume fraction of pores and tows, as well as perturbations in tow positioning or cross-section, do not significantly change the material performance for the targeted properties (under the uncertainty of the forward model). The interdependency between the tow and matrix 2-point correlations can be derived through the normalization constraints on the local material volume fractions (i.e., central peaks in 2-point auto-correlations).  

The results of this first case study demonstrate a promising framework capable of identifying interdependencies between design parameters across exceedingly high-dimensional domains. When asked to identify solutions over the regularized space of structures to match a given target property, it was not only able to identify the region in space from which the instantiating microstructure was located, but simultaneously explored the design space and identified novel microstructures. Overall, this demonstrates the ability of the framework to explore an exceptionally high-dimensional design space and provide estimate uncertainty over this domain.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_design/1556_mean_var_2pt.png" title="Posterior 2-Point Spatial Correlations" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Posterior over the set of 2-point spatial correlations computed using 1000 samples. (a) 1D cuts through the central plane X-Y of the 2-point spatial correlations with samples displayed in blue, the dashed black line corresponds to the estimated mean, and the instantiating microstructure's 2-point spatial correlations are displayed in red. (b) The mean of posterior on central X-Y plane, and (c) the variance of posterior on central X-Y plane.
</div> 

Results of the additional case studies are forthcoming in a publication titled *"Inverse Stochastic Microstructure Design"*, which will be included in this post upon publication.


References
----------

{% bibliography --file cited_posts --cited_in_order %}
