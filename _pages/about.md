---
permalink: /
title: "About Me"
excerpt: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

Welcome to my webpage!  My name is Jimmy Risk, an assistant professor of [mathematics and statistics](https://www.cpp.edu/sci/mathematics-statistics/) at [Cal Poly Pomona](https://www.cpp.edu/).  I enjoy travelling, hiking, playing pickleball, and learning Mandarin and 繁體中文.

Below you will find my contact information along with a brief introduction to my research.  You can find more detailed information by using the quick-links up top.

# Contact Information

Jimmy Risk \
Associate Professor \
Cal Poly Pomona \
3801 W Temple Ave, Pomona CA 91768\
Department of Mathematics and Statistics\
Room 8-202\
``(+1) (231) 633 1473``\
``jrisk (at) cpp (dot) edu``

# Developed and Maintained Software Packages

## EasyGPR: A User-Friendly Gaussian Process Regression Package

[EasyGPR](https://github.com/jimmyrisk/EasyGPR) is a Python package that I developed to simplify Gaussian Process Regression (GPR) for a wide range of users, especially those familiar with R. It serves as a streamlined wrapper around the GPyTorch library, offering a similar functionality and syntax to the DiceKriging package in R. EasyGPR focuses on ease of use, making Gaussian Process Regression accessible and statistically robust for various applications. This package exemplifies my commitment to bridging the gap between complex statistical methods and practical usability in research and industry.


# Primary Research Projects

## Mortality Modelling

*(Collaboration with [Mike Ludkovski (UCSB)]([https://directory.natsci.msu.edu/Directory/Profiles/Person/101627?org=55&group=87](https://ludkovski.pstat.ucsb.edu/)))* Building on my [earlier work on Gaussian process (GP) mortality models](https://www.cambridge.org/core/journals/astin-bulletin-journal-of-the-iaa/article/gaussian-process-models-for-mortality-rates-and-improvement-factors/A2D48AFF8E32CEABF9B9DB899194D9C2), recent collaborations have expanded these models to address broader mortality data challenges. Our recent work in mortality modelling involves developing a flexible GP framework for qualitative investigation of mortality data. The aim is to discover the most suitable covariance structures for mortality analysis using a genetic programming algorithm. This approach allows us to rigorously assess the presence of cohort effects and understand the relative smoothness of mortality surfaces along Age and Year dimensions. 

<div style="display: flex; justify-content: center; align-items: center;">

  <div style="flex: 1; text-align: center; padding: 10px;">
    <img src="ga.png" alt="Genetic Algorithm Illustration" style="max-width: 100%; height: auto;">
    <br>
    <em>Genetic Algorithm Illustration</em>
  </div>

  <div style="flex: 1; text-align: center; padding: 10px;">
    <img src="CircBar_JPN_Female_complex_v2.png" alt="Results for JPN Female" style="max-width: 100%; height: auto;">
    <br>
    <em>Results for JPN Female</em>
  </div>

</div>



In a different application, we utilized the GP mortality framework to obtain the [winning entry in the SOA 2021 ILEC Mortality Prediction Contest](https://www.soa.org/research/opportunities/2021-individual-life-experience-contest/), focusing on the insured population in the United States. This work demonstrates the practicality and efficiency of Gaussian processes in handling large-scale, complex mortality data, paving the way for more nuanced and accurate actuarial predictions.

## Sports Analytics and Financial Mathematics

*(Collaboration with [Albert Cohen (MSU)](https://directory.natsci.msu.edu/Directory/Profiles/Person/101627?org=55&group=87))* In the exciting intersection of sports analytics and financial mathematics, I have explored innovative frameworks for player valuation in soccer. Utilizing principles from financial mathematics and network theory, our valuation model leverages a "passing matrix" modeled by a Markov chain to encapsulate player interactions on the field, utilizing centrality measures to quantify individual influence. This approach provides a dynamic framework for ascertaining a player's fair market value, validated through a case study in the European Premier League (EPL). 

| <image src = "Liverpool--MohamadSalah_player_involvement_norm.png" width="235px" height="235px"></image> | <image src = "Liverpool--TrentAlexander-Arnold_player_involvement_norm.png" width="235px" height="235px"></image> |<image src = "Liverpool--VirgilVanDijk_player_involvement_norm.png" width="235px" height="235px"></image> |

The above images show a metric of player performance, *the normalized probability a player was involved in a play given that a shot on goal happened* (weighed to prefer successful shots), with salary superimposed.  Key scorers like Mohamad Salah illustrate generally higher normalized player involvement values, which is expected.  Next steps are to use mathematical finance tools to compute risk-neutral salaries based on this data, to compare to the realized salaries.

## Gaussian Process Super-Resolution (GPSR)

*Super-resolution* is the term of enlarging low-resolution images while restoring *high-frequency details*.  A *Gaussian process* is a specific type of model that can be used for this task.

* See the **low-resolution** image of the stairs below, whose **ground-truth** is presented next to it.  
* Two Gaussian processes are applied to this image (one with the linear kernel and one with the Laplace kernel) to attempt to restore the low-resolution image to the ground truth
  * The models are not allowed to have the ground-truth available to them!

| Low-Resolution | Ground Truth | Linear Kernel | Laplace Kernel  |
|:---:|:---:|:---:|:---:|
| <image src = "SC2_LR.png" width="219px" height="219px"></image> | <image src = "SC2_GT.png" width="219px" height="219px"></image> |<image src = "SC2_DP.png" width="219px" height="219px"></image> | <image src = "SC2_EXP.png" width="219px" height="219px"></image> |

* Our work involves *kernel analysis* for this task.
* Current research sticks with a "tried-and-true" kernel (linear, or RBF).  
* However, we find improvements in using other kernels, like the *Laplace kernel*, which specifically allows for sharp transitions in pixel intensity.
* See enlarged details below.

| Linear Kernel (Zoomed In)  | Laplace Kernel (Zoomed In) |
|---|---|
| <image src = "SC2_DP1.png" width="260px" height="120px"></image> | <image src = "SC2_EXP1.png" width="260px" height="120px"></image> |

* This work is currently on hold due to hardware failure, but the code is being refactored to work with GPyTorch, a highly efficient and modular implementation of GPs, with GPU acceleration.  This refactoring will allow for promising future research in GPSR including multi-output GPs (e.g. over color channels or patches), and sparse/variational GPs for near real-time GPSR.


