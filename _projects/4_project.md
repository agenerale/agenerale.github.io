---
layout: page
title: Bayesian Materials Processing Route Design
description: Stochastic design of materials through linking their processing route to bulk performance.
img: assets/img/inv_psp/framework.png
importance: 2
category: work
scholar:
  bibliography_template: {{reference}}
---

Inverse problems are central to material design. While numerous studies have focused on designing microstructures by inverting structure--property linkages for various material systems, such efforts stop short of providing realizable paths to manufacture such structures. Accomplishing the dual task of designing a microstructure and a feasible manufacturing pathway to achieve a target property requires inverting the complete process--structure--property linkage. However, this inversion is complicated by a variety of challenges such as inherent microstructure stochasticity, high-dimensionality, and ill-conditioning of the inversion. In this work, we propose a Bayesian framework leveraging a lightweight flow-based generative approach for the stochastic inversion of the complete process--structure--property linkage. This inversion identifies a solution distribution in the processing parameter space; utilizing these processing conditions realizes materials with the target property sets. Our modular framework readily incorporates the output of stochastic forward models as conditioning variables for a flow-based generative model, thereby learning the complete joint distribution over processing parameters and properties. We demonstrate its application to the multi-objective task of designing processing routes of heterogeneous materials given target sets of bulk elastic moduli and thermal conductivities.

**Framework**

In this section we propose a Bayesian Inversion {% cite stuart_inverse_2010 --file cited_posts %} framework for solving the process-property stochastic inverse problem. The Figure below visually summarizes the overarching strategy. To begin, we define notation specific to this particular problem. Inputs of the forward model are denote by the vector of processing parameters $$\boldsymbol{\theta} \in \mathbb{R}^D$$, the target effective property vector is denoted by $$\boldsymbol{k} \in \mathbb{R}^P$$, and intermediate (2D, in this work) microstructures are written $$m \in \mathbb{R}^{L \times L}$$. Given an initial dataset of processing parameter, microstructure, and property triples, $$\{\boldsymbol{\theta}_{n},m_{n},\boldsymbol{k}_{n}\}_{n=1}^{N}$$, we create tractable estimations for the components of Bayes' theorem necessary for performing inference (e.g., $$p(\boldsymbol{\theta})$$, $$p(\boldsymbol{k} \vert \boldsymbol{\theta})$$). 

We note that a naïve approach to identifying the mappings $$\mathcal{G}:\boldsymbol{\theta} \rightarrow m$$ and $$\mathcal{F}:m \rightarrow \boldsymbol{k}$$ would necessarily traverse the high-dimensional stochastic microstructure space. This challenge would only be amplified during the inversion. We address these issues directly by quantifying microstructures through 2-point spatial correlations, and then embedding them into a lower-dimensional manifold through Principal Component Analysis (PCA) {% cite khosravani_development_2017 latypov_data-driven_2017 paulson_reduced-order_2017 iskakov_application_2018 --file cited_posts %}, retaining $$R$$ principal components (PC) scores (denoted $$\boldsymbol{\alpha}$$) as descriptors. Importantly, this physics-informed $$R$$-dimensional representation has effectively demonstrated a robust ability of stratifying associated properties in the first handful of terms {% cite gupta_structureproperty_2015 latypov_materials_2019 yabansu_digital_2020 marshall_autonomous_2021 paulson_reduced-order_2017 generale_reduced-order_2021 kalidindi_feature_2020 --file cited_posts %}, enabling us to move through a $$R$$-dimensional microstructure space as higher-order terms result in minimal shifts in resulting properties. This distillation of global microstructure information greatly simplifies the overarching inversion, as identifying a distribution over the exceedingly high-dimensional microstructure space is wholly unnecessary. Rather, we can traverse this compact representation of the microstructure random process. However, this approach could potentially break down when the target properties are the result of localized processes, such as fracture and fatigue resistance {% cite mcdowell_microstructure-sensitive_2010 muth_neighborhood_2023 meyer_fem_2016 murakami_continuum_2012 --file cited_posts %}.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_psp/framework.png" title="Depiction detailing the overall framework." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Visual representation of proposed framework. The prior distribution in the first two dimensions is invertibly transformed to the uniform distribution, which can then be sampled to produce unique microstructure instantiations. This prior over processing parameters results in an equivalent prior in microstructure space when phase-field simulations are performed. This latent microstructure prior, alongside processing through quantification with 2-point spatial correlations and PCA embedding is also shown. The likelihood is then defined as a composition of probabilistic PS and SP linkages, which is subsequently used to perform amortized stochastic variational inference. Lastly, the image of posterior samples under the (true) PS forward model is displayed, demonstrating microstructure specificity to match target properties.
</div> 

In summary, the proposed modular framework combines surrogate forward models $$\mathcal{G}$$ and $$\mathcal{F}$$ with a flow-based generative model to explore the mapping between processing parameters and properties. 

**Experiments & Discussion**

In order to validate the proposed framework, we apply it to two different materials systems undergoing spinodal decomposition, with two distinct test cases per system. These cases consist of (processing parameters, property) pairs denoted as $$\{ \boldsymbol{\theta}^*, \boldsymbol{k}^*\}$$, which we selected from a dataset of spinodal decomposition phase-field simulations. The phase-field simulations take a 3-dimensional processing parameter input $$\boldsymbol{\theta}$$, dictating disparate heterogeneous microstructure evolution pathways. The property set $$\boldsymbol{k}$$ considered includes both effective anisotropic elastic moduli and thermal conductivity. 

The two materials systems considered were chosen to evaluate model performance with varying contrast between phases, as well as its ability to balance competing property targets. The low-contrast case corresponds to the Al-Si alloy material system, which is physically realizable. Conversely, the high-contrast case is a fictitious 2-element alloy in which the elastic and thermal properties trade off (and vary heavily) between phases. The constituent material parameters can be found in Table \ref{tab:constituent_props}. These stress tests were selected to best scrutinize the proposed approach across sparse regions of the dataset and increasingly-complex material responses.

The dataset contains 10,000 two-phase microstructures of size $$256 \times 256$$ voxels, each associated with processing parameters $$\theta_1$$ and $$\theta_2$$, sampled according to the log-uniform $$\log(\theta) \sim \mathcal{U}(\log(0.1),\log(100))$$, and $$\theta_3 \sim \mathcal{U}(-0.7,0.7)$$. Homogenized properties were extracted for each microstructure in the dataset through performing finite element simulations with Abaqus/Standard \cite{dassault_abaquscae_2019}. Part (a) visually summarizes the resulting property set for these microstructures with Al-Si constituent property assignment and part (b) with the high-contrast assignment. Part (c) displays the microstructure ensemble in the first 4 PC dimensions. The selected test cases are highlighted in each projection as $$p_1$$ and $$p_2$$.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_psp/dataset.png" title="Experimental dataset considered, with microstructure test cases highlighted." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Experimental dataset considered in this work with select microstructures identified as test cases. (a) Property set of the Al-Si microstructure ensemble, (b) property set of high-contrast microstructure ensemble, and (c) low-dimensional microstructure representation in first 4 PC dimensions.
</div> 

Part (c) of the Figure illustrates projections of the microstructure ensemble in the first 4 PCs. Each microstructure is statistically represented by a subset of its 2-point spatial correlations: $$\{f^{00}_r, f^{01}_r\}$$, with the material local state indexed by $$\{0,1\}$$. While this set of spatial correlations provides a robust statistical description of internal microstructure arrangement, it also doubles the dimensionality relative to the initial microstructure. As such, Principal Component Analysis (PCA) was performed to reduce the dataset's dimensionality, using a concatenated flattened representation of $$\{f^{00}_r, f^{01}_r\}$$. Scaling factors were applied to the set as $$\{f^{00}_r, 26.389f^{01}_r\}$$ to balance contributions between auto- and cross-correlations so that neither set dominated the transformation. Overall, Figure \ref{fig:dataset} highlights the motivation for selecting these particular test cases, with each selected to balance coverage of the property space while simultaneously identifying sparse regions in PC space.

**Al-Si Alloy**

After training, samples from the approximate posterior $$p_{n}(\boldsymbol{\theta} \vert \boldsymbol{k}^*)$$ were drawn from the CNF model for both of the test cases, previously seen in the Figure of the dataset. The marginal distributions in the log-space of the processing parameters can be seen in the Figure of the dataset, where the processing parameter set $$\boldsymbol{\theta}^*$$ associated with the conditioning property set $$\boldsymbol{k}^*$$ is demarcated with a vertical line. Inspection of the posteriors highlights that in the first case study, larger deviations in $$\theta_1$$ and $$\theta_2$$ can be tolerated, but extreme specificity is required in $$\theta_3$$, which, notably, was found to be very near to zero. This observation aligns with the fundamental dynamics of the Cahn-Hilliard phase-field model. When the order parameter $$\theta_3$$ is near zero, signifying an almost equal fraction of both phases in the initial mixture, the system experiences a heightened driving force for phase separation rendering it less sensitive to mobility parameter discrepancies in the early stages of the microstructure evolution. As a result, even significant variations in the mobility parameters, $$\theta_1$$ and $$\theta_2$$, have a diminished impact on the microstructure evolution. In contrast, the second case study posterior demonstrates significantly more sharpening across the processing parameter domain, indicating that in sparser regions of the microstructure and property spaces, a greater degree of specificity across all processing parameters is necessary to meet the target property set. In each of the marginal posterior plots, the framework was able to recover the latent $$\boldsymbol{\theta}^*$$ associated with the target property set.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_psp/marg_posterior.png" title="Experimental dataset considered, with microstructure test cases highlighted." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Marginal posterior distributions for the two test cases with the low-contrast constituent property assignment. The first two processing parameter dimensions are displayed in the log-space. Associated processing parameter sets to the target property set are displayed as a black dashed line.
</div> 

Next, we interrogate how well the posterior predictions match the desired target property set. To address this we evaluate the image of the posteriors samples under the composite surrogate forward model $$\mathcal{H}(p_{n}(\boldsymbol{\theta} \vert \boldsymbol{k}^*))$$ and compare the resulting property predictions to the initial target $$\boldsymbol{k}^*$$. As a means of validation, we simultaneously obtain $$200$$ valid samples from the posteriors and push these samples through the phase-field and finite element simulations (the ``true'' forward model). Posterior samples outside of the validity bounds of the phase-field model were rejected.  The results of this can be seen in the Figure below, where the property target falls well within the predicted distribution. Additionally, the resulting property distribution roughly matches the validation set in both variance and location, in effect validating the quality of the inversion.


**High-Contrast Alloy**

The low-contrast Al-Si test cases demonstrated that the framework is capable of adequately identifying the processing parameter sets necessary to meet various target property sets. Due to the higher contrast of the microstructures' constituent properties and increasing nonlinearity of their responses in this case study, we expect less variability in the distributions over processing parameters and latent microstructure statistics. Such intuition is reflected in the resulting posteriors $$p_{n}(\boldsymbol{\theta} \vert \boldsymbol{k}^*)$$, displayed in the Figure below. $$p_1$$ demonstrates a departure from the low-contrast log-uniform, with a preference towards lower mobility values ($$\theta_1$$ and $$\theta_2$$), while the posterior $$p_2$$ remained roughly equivalent to the prior case study due to its extremal location in the dataset.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_psp/marg_posterior_hi.png" title="Posterior of high-constrast constituient assignment test cases." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Marginal posterior distributions for the two test cases with the high-contrast constituent property assignment. The first two processing parameter dimensions are displayed in the log-space. Associated processing parameter sets to the target property set are displayed as a black dashed line.
</div> 

In a similar fashion to the prior case study, we turn towards inspecting the resulting property set predictions of the processing parameter posterior. We similarly evaluated the posteriors through the forward operator $$\mathcal{H}(p_{n}(\boldsymbol{\theta} \vert \boldsymbol{k}^*))$$, and simultaneously applied the exact version of the operator to the $$200$$ sets of processing parameters (e.g., phase-field and finite element simulations). The posterior in the property space can be seen in the Figure below, where the target property set is recovered by both the surrogate and exact operator. Interestingly, the validation distributions over $$k_1$$ and $$k_2$$ are in fact identified with smaller variance when compared to the internal predictions of the framework. We believe this is due to the comparatively worse performance of the SP linkage for thermal conductivities near the lower constituent property assignment. Importantly, even with this degradation in forward property prediction, the framework is capable of identifying promising processing parameter sets in validation.

 <div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/inv_psp/marg_pushfwd_hi.png" title="Marginal posteriors in the property space for the corresponding processing parameters." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Marginal property posterior distributions for the two high-contrast test cases shown alongside the validation set of 200 processing parameters sampled from the posteriors. The validation set was evaluated through phase-field and finite element simulations.
</div> 

Results of the additional case studies are forthcoming in a publication titled *"A Bayesian Approach to Designing Microstructures and Processing Pathways for Tailored Material Properties"*, which will be included in this post upon publication.


References
----------

{% bibliography --file cited_posts --cited_in_order %}
