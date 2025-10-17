---
layout: single
title: "MLOps Proof of Concept: Rock Paper Scissors App"
permalink: /rps/
author_profile: false
# update this when your Cloudflare Tunnel uniform resource locator (URL) rotates
api_base: "https://mlops-rps.uk"
mathjax: true
---

# RPS Quest

A game based on Rock-Paper-Scissors, **RPS Quest** is a proof of concept project that builds all steps of the MLOps lifecycle (and more) in a real environment. The system couples a FastAPI gameplay service with a live model registry to observe the complete pipeline (data acquisition, feature extraction, validation, retraining, and promotion) in real time. Full operational details are available in the [GitHub repository](https://github.com/jimmyrisk/rps) and the `docs/` directory.

You can play the game at the bottom of this page or in a [separate browser window](https://mlops-rps.uk/ui-lite).

## Game Mechanics

RPS Quest is a variant of Rock-Paper-Scissors where Rock beats Scissors, Scissors beats Paper, and Paper beats Rock. Each game runs to 10 points *(the UI displays $\text{HP} = 100 - 10\cdot\text{Points}$ for flavor)*. 

* At each round $t$, the human chooses a move $x_t \in (\mathrm{R}, \mathrm{P}, \mathrm{S})$, and 
* the bot responds with $y_t\in(\mathrm{R}, \mathrm{P}, \mathrm{S})$. 
* If both the human and the bot pick the same choice (draw), both gain 0.5 points *(to prevent drawn-out games)*.
* If there is a winner of the round, they earn points according to deterministic round multipliers $w_t = (w_t^{(R)}, w_t^{(P)}, w_t^{(S)})$. These multipliers are generated per game (lowest, middle, highest respectively):

$$w_t^{(\ell)} = 1.0, \qquad w_t^{(m)} = \beta_t, \qquad w_t^{(h)} = \beta_t + 0.5,$$

where $\beta_t \in (1.1, 1.2, 1.3, 1.4, 1.5)$ and $(\ell, m, h)$ is assigned a permutation to $(\mathrm{R}, \mathrm{P}, \mathrm{S})$ (randomly, always ensuring that that the lowest-value action beats the highest, the highest beats the middle, and the middle beats the lowest). So the most naive greedy strategy would simply pick the largest value every time (and lose once the opponent realizes the pattern is exploitable).

As an example, if the human plays Rock and the bot plays Scissors, the human earns $w_t^{(R)}$ points. If Rock had the highest points, then Paper has the lowest, and Scissors the middle.  If both play Rock, the round is a draw and each player earns 0.5 points (to prevent excessively drawn-out games). The game ends when either player reaches 10 points *(equivalently, 0 hp)*. 

The first three rounds ($t = 1, 2, 3$) follow scripted opening gambits (following what is common in [competitive rock paper scissors](https://wrpsa.com/gambits-of-rock-paper-scissors/)) to provide a unified state space for modelling, and also to provide an informed initial action distribution *(this is also generally a good strategy, look at the link!)*

### Bot Policy and State Representation

The bot policy operates on a Markovian state representation. Let $u_t$ and $b_t$ denote the human and bot cumulative scores at the start of round $t$. The state at time $t$ is
$$
  s_t = \big(x_{t-1}, x_{t-2}, x_{t-3}, \mathcal{H}_{t-1}, z_t\big),
$$
where:
- $x_{t-1}, x_{t-2}, x_{t-3}$ are the three most recent human moves,
- $\mathcal{H}_{t-1}$ aggregates historical statistics computed from rounds $1, \ldots, t-1$ (cumulative move frequencies, favored-move tendencies, lagged point values; see the next section), and
- $z_t = (u_t, b_t, w_t, t)$ are the known quantities at time $t$: current scores, round multipliers, and step index.

This structure ensures the defined state is truly Markovian: $\mathcal{H}_{t-1}$ depends only on past information, while $z_t$ contains all time-$t$ observables before the human chooses $x_t$.

## Feature Engineering

The specific features stored in $s_t$ are described below.  Categorical variables are one-hot encoded, resulting in a total of 50 features.

| Feature Group | Count | Description |
|:--------------|:-----:|:------------|
| User move history | 9 | One-hot encoding of $x_{t-1}, x_{t-2}, x_{t-3}$ |
| Bot move history | 9 | One-hot encoding of $y_{t-1}, y_{t-2}, y_{t-3}$ |
| Result history | 9 | One-hot encoding of round outcomes (win/lose/draw) for $t-1, t-2, t-3$ |
| Current round points | 3 | $w_t^{(R)}, w_t^{(P)}, w_t^{(S)}$ (from $z_t$) |
| Lagged point values | 9 | Historical point weights from previous three rounds (from $\mathcal{H}_{t-1}$) |
| User tendencies | 3 | From $\mathcal{H}_{t-1}$: proportion of time user picks $R$, $P$, or $S$ |
| Favored-move tendencies | 3 | From $\mathcal{H}_{t-1}$: proportion of time the user picks (i) the highest-value move, (2) its counter, or (3) counter-counter |
| Score context | 4 | From $z_t$: $u_t - b_t$ (score differential), $10-u_t$ (points-to-win for user), $10-b_t$ (points-to-win for bot), step number $t$ |
| Legacy flag | 1 | easy_mode (deprecated; always 0 in new games) |



Using this setup, we can produce a multinomial distribution over the user's move for this round:

$$\mathbf{p}_t(s_t) = \big(p_t^{(R)}, p_t^{(P)}, p_t^{(S)}\big) = \mathbb{P}(x_{t} \mid s_t).$$

Remark: Before training, features are standardized using scikit-learn's `StandardScaler` fitted on the training set. The same scaler parameters are serialized with each model to ensure inference applies identical transformations.

## Model Training & Loss Functions

Model training minimizes the negative log-likelihood (cross-entropy loss) over historical gameplay data. For a batch of labeled examples $(s_i, x_i)$, the standard objective is

$$\mathcal{L}_{\text{CE}} = -\frac{1}{N}\sum_{i=1}^{N} \log p_{\theta}^{(x_i)}(s_i),$$

where $\theta$ denotes model parameters and $p_{\theta}^{(x_i)}(s_i)$ is the predicted probability of the true move $x_i$.  Note that this can be augmented with various penalties, like for logistic regression we include a ridge penalty.  For a game specific penalty, we include a customized **danger penalty** $\lambda_{\text{danger}}$. In this case,

$$\mathcal{L} = \mathcal{L}_{\text{CE}} + \lambda_{\text{danger}} \cdot \frac{1}{N}\sum_{i=1}^{N} p_{\theta}^{(d(x_i))}(s_i),$$

where $d(x_i)$ is the *danger class* for each move (the one that beats it):

$$d(R) = P, \quad d(P) = S, \quad d(S) = R.$$

The penalty term $\frac{1}{N}\sum_i p_{\theta}^{(d(x_i))}$ is the mean probability mass the model places on the move that would make the bot lose. In contrast with typical regularization penalties, higher $\lambda_{\text{danger}}$ leads the model toward thinking one step ahead to beat an anticipatory player. The feedforward neural network model also applies a post-hoc adjustment at inference time: it multiplies the danger class probability by $e{-\lambda_{\text{danger}}}$ and renormalizes, further suppressing risky outputs.

## Decision Making: Bellman Optimality and Greedy Decisions

To make actual in-game decisions, each model uses a fixed policy rule assuming the probabilities it utilizes are the truth for this round.  Mathematically, the underlying control problem involves a finite-horizon Markov decision process with a terminal condition at $\tau = 10$ points. An optimal controller would solve the discrete Bellman optimality equation backward in time:

$$V_t(s) = \max_{a \in \mathcal{A}} \Big[ r_t(s,a) + \gamma \, \mathbb{E}\big[ V_{t+1}(S_{t+1}) \mid s, a \big] \Big],$$

with $V_T(s) = 0$ once either player hits the target score $\tau$ and discount $\gamma = 1$. Computing this exactly requires enumerating full joint trajectories over $s_t$.  In practice, this is doable with a short horizon, but due to potential concerns with server issues and runtime, we utilize an approximation.

In particular, the gameplay service uses a greedy, one-step surrogate. For any candidate bot move $a$, let $b(a)$ denote the human action beaten by $a$ and $\ell(a)$ the action that defeats $a$. The approximation computes

$$\widehat{Q}_t(a) = p_t^{(b(a))} \Big(w_t^{(a)} + B_t(a)\Big) - p_t^{(\ell(a))} \Big(w_t^{(\ell(a))} + L_t(a)\Big) + p_t^{(a)}\, C_t,$$

where

$$B_t(a) = \mathbf{1}(u_t + w_t^{(a)} \ge \tau)\cdot \tau, \quad L_t(a) = \mathbf{1}(b_t + w_t^{(\ell(a))} \ge \tau)\cdot \tau, \quad C_t = \tfrac{1}{2}\,\frac{u_t - b_t}{\tau}.$$

The policy picks $\arg\max_a \widehat{Q}_t(a)$. The first term is the expected reward if the human plays the move $a$ beats. The second is the penalty if they counter. The third is a tie-breaker that respects the current score differential. This greedy heuristic, backed by the probabilistic predictions, matches the qualitative behavior of a full dynamic program in offline rollouts while remaining cheap enough to execute in the live API.


## Model Roster and MLOps Technicalities

For the currently deployed app, there are three production model types are maintained, each with a Production alias, a B-test alias, and two shadow slots for staging promotions (a total of 12 models):

- **Feedforward neural network ("Brian")** — A PyTorch architecture with batch normalization, dropout, and ReLU activations. Trained with the danger penalty in the loss function and applies an additional post-inference adjustment to suppress risky predictions. Hidden layer configurations range from compact (64, 32) to wide (512, 256) across the hyperparameter sweep. This model's flexibility allows it to learn complex nonlinear patterns in the 50-feature space.

- **XGBoost ("Forrest")** — Gradient-boosted decision trees trained on the same 50 features. Tree-based models handle feature interactions naturally and provide interpretable feature importance scores. Game-stratified folds and careful tuning of tree depth, learning rate, and number of estimators ensure robust predictions. XGBoost does not use the danger penalty directly (as it outputs class probabilities via softmax), but the greedy policy downstream evaluates these probabilities conservatively.

- **Multinomial logistic regression ("Logan")** — A single-layer softmax model with optional class weighting and the danger penalty integrated into the loss. This model has limitations, as we do not expect human decisions to be linear over a collection of straightforward features. However, it provides a calibrated probabilistic baseline, and still provides a challenge.

Hyperparameter sweeps explore architectures, learning rates, dropout rates, weight decay, $\lambda_{\text{danger}}$, ridge regularizations, and more. Each configuration trains with game-stratified $k$-fold cross-validation to respect timing issues boundaries and prevent data leakage. After training, models are logged to (MLflow)[https://dagshub.com/jimmyrisk/rps/models]. A CronJob orchestrator evaluates cross-validation accuracy and assigns models to Production, B, shadow1, or shadow2 aliases. The auto-promotion logic swaps aliases when a challenger outperforms the "production" model on live testing games.

All three trainers share the same trainer pipeline via MLflow, feature extraction and scaling, and game-stratified cross-validation. Each model produces $\mathbf{p}_t$ at inference time by calling the trained model on $s_t$ (with models loaded in a cache). CronJobs in the cluster automate the process by rotating aliases, evaluate B-versus-Production win rates, and train models continuously to new data.

## More Links

- Play the Game: [mlops-rps.uk/ui-lite](https://mlops-rps.uk/ui-lite) (and the [debug interface](https://mlops-rps.uk/ui-lite-debug))
- Inspect live metrics: [Grafana dashboard](https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95)
- Explore the code and runbooks: [github.com/jimmyrisk/rps](https://github.com/jimmyrisk/rps)
- MLflow experiment lineage: [dagshub.com/jimmyrisk/rps](https://dagshub.com/jimmyrisk/rps)

Or, play the game in the embedded window below!

Have any questions?  Feel free to e-mail me or connect on linkedin!


{: .text-center}

<iframe
  src="{{ page.api_base }}/ui-lite?api_base={{ page.api_base }}"
  title="RPS Quest Lite"
  style="width:110%;min-height:780px;border:1px solid #d0d7de;border-radius:12px;"
  loading="lazy"
  allowfullscreen
>
</iframe>
