---
permalink: /
title: "About Me"
excerpt: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

Welcome to my webpage!  My name is Jimmy Risk, an assistant professor of [mathematics and statistics](https://www.cpp.edu/sci/mathematics-statistics/) at [Cal Poly Pomona](https://www.cpp.edu/).  I am learning Mandarin and 繁體中文.  I enjoy travelling, hiking, and playing tennis.

Below you will find my contact information along with a brief introduction to my research.  You can find more detailed information by using the quick-links up top.

# Contact Information

Jimmy Risk, Assistant Professor\
Cal Poly Pomona, \
3801 W Temple Ave, Pomona CA 91768\
Department of Mathematics and Statistics\
Room 8-202\
``(+1) (231) 633 1473``\
``jrisk (at) cpp (dot) edu``



# Primary Research Projects

## Multi-Population Mortality Models

Having [published work in Gaussian process (GP) mortality models](https://www.cambridge.org/core/journals/astin-bulletin-journal-of-the-iaa/article/gaussian-process-models-for-mortality-rates-and-improvement-factors/A2D48AFF8E32CEABF9B9DB899194D9C2), my colleagues have continued working with [GP's in multi-population settings](https://www.cambridge.org/core/journals/annals-of-actuarial-science/article/abs/multioutput-gaussian-processes-for-multipopulation-longevity-modelling/8D169937E25A576B1D39CD792F57B117).  Roughly speaking, the idea is to group "populations" together to model them together as opposed to individually.  Below we provide forecasted plots on four populations, UK Males, UK Females, Japan Males, Japan Females, for individuals age 84 to project mortality improvement factors.

| Japan Male | Japan Female | UK Male | UK Female  |
|---|---|---|---|
| <image src = "japmale9.png" width="219px" height="156px"></image> | <image src = "japfemale9.png" width="219px" height="156px"></image> |<image src = "ukmale9.png" width="219px" height="156px"></image> | <image src = "ukfemale9.png" width="219px" height="156px"></image> |

A way to combine the populations can be done through the [Linear Model of Coregionalization (LMC)](https://arxiv.org/pdf/1106.6251.pdf), as is done in my colleagues work.  However, multiple more recent multi-output GP frameworks can be considered, for example the [Cross Spectral Mixture](https://dl.acm.org/doi/abs/10.5555/2969442.2969463) and [Multi Output Spectral Mixture](https://arxiv.org/pdf/1709.01298.pdf).  Additionally, individual kernels can be specified for the LMC to express certain data properties, to improve model tractability and interpretability.

## Gaussian Process Super-Resolution

*Super-resolution* is the term of enlarging low-resolution images while restoring *high-frequency details*.  A *Gaussian process* is a specific type of model that can be used for this task.

* See the **low-resolution** image of the stairs below, whose **ground-truth** is presented next to it.  
* Two Gaussian processes are applied to this image (one with the linear kernel and one with the Laplace kernel) to attempt to restore the low-resolution image to the ground truth
  * The models are not allowed to have the ground-truth available to them!

| Low-Resolution | Ground Truth | Linear Kernel | Laplace Kernel  |
|---|---|---|---|
| <image src = "SC2_LR.png" width="219px" height="219px"></image> | <image src = "SC2_GT.png" width="219px" height="219px"></image> |<image src = "SC2_DP.png" width="219px" height="219px"></image> | <image src = "SC2_EXP.png" width="219px" height="219px"></image> |

* Our work involves *kernel analysis* for this task.
* Current research sticks with a "tried-and-true" kernel (linear, or RBF).  
* However, we find improvements in using other kernels, like the *Laplace kernel*, which is intended to allow for sharp transitions in pixel intensity.
* See improved details below.

| Linear Kernel (Zoomed In)  | Laplace Kernel (Zoomed In) |
|---|---|
| <image src = "SC2_DP1.png" width="260px" height="120px"></image> | <image src = "SC2_EXP1.png" width="260px" height="120px"></image> |


## Connections between Neural Networks and Gaussian Process Kernels

[Recent work](https://arxiv.org/pdf/1806.07572.pdf) has shown a type of convergence of neural networks to other kernel regressors (e.g. Gaussian processes).  [Later work](https://arxiv.org/pdf/2009.10683.pdf) has shown specific convergence to the Laplace kernel. My student Ronald Lencevicius and I are investigating emperical results to understand rates of convergence and generalization to data sets.

<center><img src="nn.PNG"></center>

The result relies on the kernel regressor (e.g. posterior mean of GP) and neural network belonging to the same reproducing kernel Hilbert space (RKHS).  However, the underlying Gaussian process does not belong to the same RKHS as its posterior mean.  Current plans are to investigate Gaussian process draws and the neural network draws (not expected values) to assess possible connections, for example regarding uncertainty estimation.



