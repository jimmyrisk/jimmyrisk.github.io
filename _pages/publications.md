---
layout: archive
title: "Research"
permalink: /publications/
author_profile: true
---

{% if author.googlescholar %}
  You can also find my articles on <u><a href="{{author.googlescholar}}">my Google Scholar profile</a>.</u>
{% endif %}

{% include base_path %}

# Research Interests

My research primarily focuses on **Gaussian processes** (GPs), a Bayesian non-parametric based modelling method.  I have used GPs as a tool in a variety of disciplines including

* **financial mathematics** (quantile of loss estimation and pricing), 
* **actuarial science** (mortality modelling and pricing), 
* **modern machine learning tasks** (super-resolution, computer vision classification) 

Gaussian processes are a kernel based method (the same kernel used in e.g.~support vector machines) and much of my current research is on analyzing and interpreting various roles of kernels in machine learning tasks.  A primary goal of mine is to connect interpretable and well founded methodology in probability and statistics to modern data science.  For example, a neural network used to model an unknown mapping ``f`` is a black box in the sense that studying the neural network will give limited to no insight on the structure of the function being approximated.  In contrast, in using a GP for this task, the kernel of a GP yields the underlying properties of ``f``, such as differentiability, continuity, periodicity, etc..  

I acknowledge state of the art problems and methodology, and value a probabilistic and statistical approach to make advanced tasks more transparent, interpretable, and robust.

# Publications

* Risk, Jimmy, Huynh, Nhan, and Ludkovski, Michael.  **SOA 2021 ILEC mortality prediction contest.**  *Society of Actuaries* (2021). [www.soa.org/globalassets/assets/files/resources/research-report/2021/mort-prediction-contest.pdf](https://www.soa.org/globalassets/assets/files/resources/research-report/2021/mort-prediction-contest.pdf)
* Risk, Jimmy, and Ludkovski, Michael. **Sequential Design and Spatial Modeling for Portfolio Tail Risk Measurement.** *SIAM Journal on Financial Mathematics 9.4* (2018) 1137-1174. *([arxiv link](https://arxiv.org/pdf/1710.05204.pdf))*
* Ludkovski, Michael, Risk, Jimmy, and Zail, Howard. **Gaussian Process Models for Mortality Rates and Improvement Factors.** *ASTIN Bulletin: The Journal of the IAA 48.3*  (2018) 1307-1347. *([arxiv link](https://arxiv.org/pdf/1608.08291.pdf))*
  * Accompanied ``R`` Notebook: [github.com/jimmyrisk/GPmortalityNotebook](https://github.com/jimmyrisk/GPmortalityNotebook)
* Risk, Jimmy, and Ludkovski, Michael. **Statistical emulators for pricing and hedging longevity risk products.** *Insurance: Mathematics and Economics* 68 (2016): 45-60. *([arxiv link](https://arxiv.org/pdf/1508.00310.pdf))*
* Risk, Jimmy.  **Correlations between Google search data and Mortality Rates.** arXiv preprint arXiv:1209.2433 (2012).  [https://arxiv.org/abs/1209.2433](arxiv.org/abs/1209.2433)


# Working Papers

* Risk, Jimmy, Amelin, Charles, and Frank, Hakeem.  **Interpretable Kernels for Gaussian Process Super-Resolution.** *(Preprint available upon request)*
* Risk, Jimmy, and Ronald Len\v{c}evicius.  **Gaussian Process Specific Comparisons between Neural Tangent and Laplace Kernels.**
* Risk, Jimmy, and Ludkovski, Michael.  **Flexible Kernels for Multi-Population Gaussian Process Mortality Models.**
