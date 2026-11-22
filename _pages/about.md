---
layout: about
title: about
permalink: /
subtitle: <strong>Scientific Machine Learning • Materials Informatics</strong>

profile:
  align: right
  image: Picture1.jpg
  image_circular: false # crops the image to make it circular
  address: >
    <!---<p>555 your office number</p>
    <p>123 your address street</p>
    <p>Your City, State 12345</p>--->

news: false  # includes a list of news items
selected_papers: true # includes a list of papers marked as "selected={true}"
social: true  # includes social icons at the bottom of the page
---

I lead a machine learning group at Pratt & Whitney working on probabilistic models of gas turbine hot-section components. Current problems include calibrating multiphysics damage models to sparse field and inspection data through stochastic variational inference and Bayesian optimization; modeling component removals with Bayesian survival models over multimodal, right-censored fleet data; and approximating coupled thermo-mechanical fields with neural operators that enforce conservation of energy and momentum. The same probabilistic models extend to design, where conditional generative models sample candidate designs conditioned on performance and life requirements, and to agentic systems, in which language models plan multi-step analyses and invoke these surrogates, differentiable solvers, and inference routines as tools.

My research, at the intersection of scientific machine learning and materials informatics, concerns inference and design in high-dimensional, stochastic, partially observed systems. A recurring approach is to represent conditional densities with transport maps, learned with continuous normalizing flows or flow matching and amortized over conditioning variables, so that Bayesian inversion remains tractable when likelihoods are expensive or implicit. Recent examples include conditional variable flow matching for stochastic dynamics observed through sparse, unpaired measurements and flow-based estimation of microstructure-sensitive exceedance distributions in unidirectional polymer-matrix composites.

My doctoral work at Georgia Tech (Ph.D., 2024), advised by [Surya R. Kalidindi](https://www.me.gatech.edu/faculty/kalidindi) in the [MINED Research Group](https://mined.gatech.edu/), developed Bayesian frameworks for the stochastic inversion of process–structure–property linkages, combining microstructure priors built on n-point spatial statistics, Gaussian-process surrogate likelihoods, and learned posteriors over microstructures and the processing conditions that realize them. Earlier, I designed cooled turbine airfoils and ceramic matrix composite components at Pratt & Whitney, work that resulted in more than twenty U.S. patents.
