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

*I am currently seeking opportunities in Data Science Engineer and Machine Learning Engineer roles. Please feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/jimmy-risk-02b46330/).*

# Contact Information

Jimmy Risk \
Associate Professor \
Cal Poly Pomona \
3801 W Temple Ave, Pomona CA 91768\
Department of Mathematics and Statistics\
Room 8-202\
``(+1) (231) 633 1473``\
``jrisk (at) cpp (dot) edu``

# Recent Publication

<img src="book.webp" alt="Gaussian Process Models for Quantitative Finance book cover" width="200" style="float: right; margin-left: 20px; margin-bottom: 10px;">

My co-authored book **Gaussian Process Models for Quantitative Finance** (with [Dr. Michael Ludkovski, UC Santa Barbara](https://www.pstat.ucsb.edu/people/michael-ludkovski)) has been published by Springer in the SpringerBriefs in Quantitative Finance series. This book provides the first comprehensive treatment of Gaussian Processes in finance, including extensive literature reviews, advanced methodologies, theoretical foundations, and computational strategies. It serves as a vital resource for researchers and practitioners working at the intersection of machine learning and quantitative finance. [Available now from Springer](https://link.springer.com/book/10.1007/978-3-031-80874-6).

# MLOps Proof of Concept: RPS Quest

[RPS Quest](https://mlops-rps.uk/ui-lite) is a complete MLOps implementation disguised as a Rock-Paper-Scissors game. The project demonstrates the full machine learning lifecycle—data acquisition, feature engineering, model training, evaluation, and automated deployment—in a live production environment. Three distinct model architectures (feedforward neural networks, XGBoost, and logistic regression) compete in an automated registry, with live A/B testing determining promotion to production. The system tracks real-time performance metrics through a [Grafana dashboard](https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95), with all code and documentation available on [GitHub](https://github.com/jimmyrisk/rps) and experiment lineage tracked via [DagshHub](https://dagshub.com/jimmyrisk/rps). This project showcases practical experience with containerized microservices, CI/CD pipelines, model registries, and production monitoring—skills directly applicable to industry ML engineering roles. More details are available on the [dedicated RPS page](/rps/).


# Research Interests and Projects


### Recent Preprint: Dynamics of Liquidity Surfaces in Uniswap v3

*(Collaboration with [Dr. Shen-Ning Tung (National Tsing Hua University)](https://www.nthu.edu.tw/) and [Dr. Tai-Ho Wang (Baruch College)](https://baruch.cuny.edu/))* 

Our recent preprint presents the first comprehensive empirical analysis of liquidity dynamics in Uniswap v3, the largest decentralized exchange. Using **Gaussian Processes (GPs)**, we model liquidity surfaces across price levels and time, capturing complex spatiotemporal patterns in concentrated liquidity provision. The study analyzes multiple liquidity pools (ETH-USDC, WBTC-USDC, USDC-USDT) at different fee tiers, examining how liquidity provider behavior impacts market efficiency and stability. Our GP-based framework provides robust uncertainty quantification and demonstrates superior performance compared to traditional parametric models in forecasting liquidity trends. [Read the preprint on arXiv](https://arxiv.org/abs/2509.05013).


---

#### Stochastic Modeling in Sports Analytics and Financial Applications

*(Collaboration with [Dr. Albert Cohen (Michigan State University)](https://www.msu.edu/) and [Dr. Tai-Ho Wang](https://baruch.cuny.edu/))* 

Building on the **Pythagorean expectation** framework originally developed by Bill James, this project develops dynamic models that integrate mathematical finance with sports analytics that can be used to rigorously price financial derivates in betting markets.

**Key Focus Areas:**
- **Dynamic Pythagorean Models:** Extending the traditional Pythagorean expectation to a dynamic setting, allowing parameters to vary over time to capture evolving team performance.
- **Barrier Options in Sports Betting:** Exploring financial derivatives analogous to barrier options, where payoffs depend on performance metrics crossing specific thresholds. This innovation provides new opportunities in the sports betting market.
- **Model Improvements:** Addressing observed inconsistencies in existing models by incorporating decreasing variance and handling large shocks in performance metrics. Developing stochastic differential equation (SDE) systems to enhance model accuracy and tractability for derivative pricing.

---

#### Optimal Control in Dynamic Sports Models

*(Collaboration with [Dr. Albert Cohen](https://www.msu.edu/) and [Dr. Tai-Ho Wang](https://baruch.cuny.edu/))* 

Building upon our work in sports analytics, this project applies **stochastic control theory** to optimize team performance strategies over a season. By treating the Pythagorean exponent \( \gamma_t \) as a dynamic control variable, we develop models that allow teams to adjust their strategies in real-time to achieve desired performance outcomes. This approach provides a mathematical foundation for decision-making processes in team management, balancing consistency and adaptability.  This research is the first to bridge theoretical control models and practical sports management relating to the seminal work by Bill James, offering actionable insights for coaches and team managers.

**Model Development**  
We construct a system of stochastic differential equations (SDEs) that govern the dynamics of team performance metrics, incorporating control variables that represent strategic adjustments. The optimal control distribution is derived to minimize a value function that captures performance goals and strategic costs that can be chosen by managers and coaches.

---

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


