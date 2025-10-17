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

RPS Quest is a variant of Rock-Paper-Scissors where Rock beats Scissors, Scissors beats Paper, and Paper beats Rock. Each game runs to 10 points. At each round \(t\), the human chooses a move \(x_t \in \{\mathrm{R}, \mathrm{P}, \mathrm{S}\}\) and the bot responds with \(y_t\). The winner of the round earns points according to deterministic round multipliers \(w_t = (w_t^{(R)}, w_t^{(P)}, w_t^{(S)})\). These multipliers are generated per game and round via hash:
\[
  w_t^{(\ell)} = 1.0, \qquad w_t^{(m)} = b_t, \qquad w_t^{(h)} = b_t + 0.5,
\]
where \(b_t \in \{1.1, 1.2, 1.3, 1.4, 1.5\}\) and \(\{\ell, m, h\}\) is a permutation of moves determined by `deterministic_round_points`. For instance, if the human plays Rock and the bot plays Scissors, the human earns \(w_t^{(R)}\) points. If both play Rock, the round is a draw and no points are awarded. The game ends when either player reaches 10 points. The UI displays $\text{HP} = 100 - 10\cdot\text{Points}$ for flavor.

The first three rounds (\(t = 1, 2, 3\)) follow scripted opening gambits (following what is common in [competitive rock paper scissors](https://wrpsa.com/gambits-of-rock-paper-scissors/)) to collect an initial action distribution and yield a unified state space for modeling.

### Bot Policy and State Representation

The bot policy operates on a Markovian state representation. Let \(u_t\) and \(b_t\) denote the human and bot cumulative scores at the start of round \(t\). The state at time \(t\) is
\[
  s_t = \big(x_{t-1}, x_{t-2}, x_{t-3}, \mathcal{H}_{t-1}, \Omega_t\big),
\]
where:
- \(x_{t-1}, x_{t-2}, x_{t-3}\) are the three most recent human moves,
- \(\mathcal{H}_{t-1}\) aggregates historical statistics computed from rounds \(1, \ldots, t-1\) (cumulative move frequencies, favored-move tendencies, lagged point values),
- \(\Omega_t = (u_t, b_t, w_t, t)\) are the deterministic, known quantities at time \(t\): current scores, round multipliers, and step index.

This structure ensures the state is truly Markovian: \(\mathcal{H}_{t-1}\) depends only on past information, while \(\Omega_t\) contains all time-\(t\) observables before the human chooses \(x_t\).

## Feature Engineering

The system maintains a 50-feature contract \(\phi(s_t) \in \mathbb{R}^{50}\) implemented in `app.features` that stays identical for training and inference. Categorical variables (moves, results) are one-hot encoded. Below is the full breakdown:

| Feature Group | Count | Description |
|:--------------|:-----:|:------------|
| User move history | 9 | One-hot encoding of \(x_{t-1}, x_{t-2}, x_{t-3}\) |
| Bot move history | 9 | One-hot encoding of \(y_{t-1}, y_{t-2}, y_{t-3}\) |
| Result history | 9 | One-hot encoding of round outcomes (win/lose/draw) for \(t-1, t-2, t-3\) |
| Current round points | 3 | \(w_t^{(R)}, w_t^{(P)}, w_t^{(S)}\) (from \(\Omega_t\)) |
| Lagged point values | 9 | Historical point weights from previous three rounds (from \(\mathcal{H}_{t-1}\)) |
| User tendencies | 3 | Cumulative frequencies from \(\mathcal{H}_{t-1}\): \(\frac{\#\{R\}}{\text{total}}\), \(\frac{\#\{P\}}{\text{total}}\), \(\frac{\#\{S\}}{\text{total}}\) |
| Favored-move tendencies | 3 | From \(\mathcal{H}_{t-1}\): fraction of time user picks the highest-value move, its counter, or counter-counter |
| Score context | 4 | From \(\Omega_t\): \(u_t - b_t\), points-to-win for user, points-to-win for bot, step number \(t\) |
| Legacy flag | 1 | `easy_mode` (deprecated; always 0 in new games) |

Given \(\phi(s_t)\), each model produces a categorical distribution over the next user move:
\[
  \mathbf{p}_t = \big(p_t^{(R)}, p_t^{(P)}, p_t^{(S)}\big) = \mathbb{P}(x_{t+1} \mid s_t),
\]
which drives both the greedy policy and telemetry logged to Grafana Cloud.

## Model Roster

Three production model types are maintained, each with a Production alias, a B-test alias, and two shadow slots for staging promotions:

- **Feedforward neural network ("Brian")** — A PyTorch architecture with dropout, batch normalization, and a danger penalty \(\lambda_{\text{danger}}\) that suppresses high-risk predictions. The danger term penalizes the move that would beat the model's top prediction; if the user plays that counter-move, the bot loses. Trained with time-series cross-validation and logged via MLflow PyFunc.

- **XGBoost ("Forrest")** — Gradient-boosted trees work well on structured feature vectors. Game-stratified folds and feature importance plots check stability before promoting a new checkpoint.

- **Multinomial logistic regression ("Logan")** — A calibrated softmax model with optional class weights and the same \(\lambda_{\text{danger}}\) penalty. This provides an interpretable baseline for benchmarking the other two.

All three trainers share the same pipeline in `trainer/base_model.py`, so scalers, stratification, and artifact logging stay consistent. CronJobs in the cluster rotate aliases, evaluate B-versus-Production win rates, and push new checkpoints without manual intervention.

## From HJB to Greedy Value Control

The underlying control problem is a finite-horizon Markov decision process with a terminal condition at \(\tau = 10\) points. An optimal controller would solve the discrete-time Hamilton-Jacobi-Bellman (HJB) equation backward in time:
\[
  V_t(s) = \max_{a \in \mathcal{A}} \Big[ r_t(s,a) + \gamma \, \mathbb{E}\big[ V_{t+1}(S_{t+1}) \mid s, a \big] \Big],
\]
with \(V_T(s) = 0\) once either player hits the target score \(\tau\) and discount \(\gamma = 1\). Computing this exactly requires enumerating full joint trajectories over \(\phi(s_t)\), which is prohibitive online.

Instead, the gameplay service uses a greedy, one-step surrogate implemented in `value_optimizer_with_policy`. For any candidate bot move \(a\), let \(b(a)\) denote the human action beaten by \(a\) and \(\ell(a)\) the action that defeats \(a\). The approximation computes
\[
  \widehat{Q}_t(a) = p_t^{(b(a))} \Big(w_t^{(a)} + B_t(a)\Big) - p_t^{(\ell(a))} \Big(w_t^{(\ell(a))} + L_t(a)\Big) + p_t^{(a)}\, C_t,
\]
where
\[
  B_t(a) = \mathbf{1}\{u_t + w_t^{(a)} \ge \tau\}\cdot \tau, \quad
  L_t(a) = \mathbf{1}\{b_t + w_t^{(\ell(a))} \ge \tau\}\cdot \tau, \quad
  C_t = \tfrac{1}{2}\,\frac{u_t - b_t}{\tau}.
\]
The policy picks \(\arg\max_a \widehat{Q}_t(a)\). The first term is the expected reward if the human plays the move \(a\) beats. The second is the penalty if they counter. The third is a tie-breaker that respects the current score differential. This greedy heuristic, backed by the probabilistic predictions, matches the qualitative behavior of a full dynamic program in offline rollouts while remaining cheap enough to execute in the live API.

## Operations

The full stack runs on a single-node k3s cluster (4 GB RAM, 8 GB swap). FastAPI serves the gameplay endpoints, SQLite persists game state, MinIO caches the twelve live model aliases, and DagsHub MLflow handles experiment tracking. Deployment uses straightforward manifests (no Terraform). The deployment scripts and telemetry automation are documented in the repository README. The Grafana dashboard JSON is published via `ops/deploy_clean_dashboard.sh` and surfaced publicly for anyone to inspect.

## Try the System

- Play the Game: [mlops-rps.uk/ui-lite](https://mlops-rps.uk/ui-lite) (and the [debug interface](https://mlops-rps.uk/ui-lite-debug))
- Inspect live metrics: [Grafana dashboard](https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95)
- Explore the code and runbooks: [github.com/jimmyrisk/rps](https://github.com/jimmyrisk/rps)
- MLflow experiment lineage: [dagshub.com/jimmyrisk/rps](https://dagshub.com/jimmyrisk/rps)

{: .text-center}

<iframe
  src="{{ page.api_base }}/ui-lite?api_base={{ page.api_base }}"
  title="RPS Quest Lite"
  style="width:110%;min-height:780px;border:1px solid #d0d7de;border-radius:12px;"
  loading="lazy"
  allowfullscreen
>
</iframe>
