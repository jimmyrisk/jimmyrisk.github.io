---
layout: single
title: "MLOps Proof of Concept: Rock Paper Scissors App"
permalink: /rps/
author_profile: false
# update this when your Cloudflare Tunnel uniform resource locator (URL) rotates
api_base: "https://mlops-rps.uk"
mathjax: true
---

# Rock–Paper–Scissors



(NEED TO MENTION FEATURE NORMALIZING ETC)

As an academic aspiring to get into industry, I realized I had to beef up my skills beyond academic research and kaggle datasets.  What better way than to build a MLOps enterprise from scratch?  

In three weeks I was able to create RPS Quest, a hands-on MLOps proof of concept by an academic aspiring to get into industry!  Building this helped me own the full lifecycle: modeling, infrastructure, deployment, and observability, and more importantly, how it operates in a real-world setting.

Try it yourself! https://mlops-rps.uk/ui-lite

🧮 Gameplay runs to 10 points with deterministic round multipliers. Players choose a model to play against:
* 🧠 Brian - feedforward neural network
* 🌲 Forrest - XGBoost
* 🪵 Logan - multinomial logistic regression

🔧 How it works: 
* I treat it as a Markov decision process: state = three action lookback.   
* Each model is trained on a schedule to data ingested in real time as gameplay ensues.  
* Four live aliases (Production, B, shadow1, shadow2) per model type, each with differing hyperparameter configurations. 
* Players play against Production or B (50/50 split) to emulate A/B testing (monitor game win rates), and shadow models assess accuracies of potential models.
* Models are promoted as per fixed rules (win rates, prediction accuracy)

🧭 Practical Infrastructure: 
* a single-node k3s Kubernetes cluster orchestrates the workloads, 
* Python stack with FastAPI powering the gameplay API and XGBoost and PyTorch for modelling
* DagsHub/MLflow handle model registry + experiment tracking,
* SQLite stores game state on disk, 
* MinIO as a low-latency model cache,
* Released as Docker images to unify production with self-testing.

⚙️ Operations I now automate end-to-end:
* CronJobs trigger sequential training runs that respect a 4 GB RAM / 8 GB swap budget, syncing artifacts to MinIO and DagsHub.  (Hardware is light since I bought this server solely for this project!)
* A 50-feature contract (lagged moves, historical round values, score context) stays identical between training and serving; parity harnesses replay live games to catch drift.
* Grafana Cloud dashboards show action accuracy, game win rates, training outcomes, and promotion decisions. 
* Auto-promotion compares Production vs. B once each alias has 3 or more completed games
* Periodically rearranges B, shadow1, shadow2 according to accuracy

📈 If headcount were to grow, I'd make some changes:
* Infrastructure: adjust SQLite to PostgreSQL, add Kafka for event sourcing, layer Spark for feature windows and better data handling, and wrap it all in GitOps with autoscaling. 
* ML Side: Broader hyperparameter sweeps, add value-aware objectives so policies balance win probability with point preservation, stratified checks by user profile keeps promotion decisions honest as the player base grows.

🗒️ Interesting notes:
* Logistic regression noticably has the worst accuracy and win rate.  This makes sense as player behavior is highly complex and nonlinear.
* I was able to include a "danger" penalty in the loss function to hedges against the most likely choice in the loss function; this increased win rates for to help the neural network and logistic regression models.
* XGBoost and Logistic models provide interpretable results, basing decisions on current point values and player tendencies


Try it yourself!

* Play the Game: [mlops-rps.uk/ui-lite](https://mlops-rps.uk/ui-lite)  ([debug version](https://mlops-rps.uk/ui-lite-debug))
* View Grafana Metrics: [grafana metrics](https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95)
* Read More: [jimmyrisk.github.io/rps](https://jimmyrisk.github.io/rps/)
* Github URL: [github.com/jimmyrisk/rps](https://github.com/jimmyrisk/rps)
* Dagshub URL: [dagshub.com/jimmyrisk/rps](https://dagshub.com/jimmyrisk/rps)


{: .text-center}

<iframe
  src="{{ page.api_base }}/ui-lite?api_base={{ page.api_base }}"
  title="RPS Quest Lite"
  style="width:110%;min-height:780px;border:1px solid #d0d7de;border-radius:12px;"
  loading="lazy"
  allowfullscreen
>
</iframe>
