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
{: .text-center}



Looking at job ads, I realized I had to beef up my skills beyond the academic research and kaggle datasets.  What better way than to build a MLOps enterprise from scratch?  Building this helped me own the full lifecycle: modeling, infrastructure, deployment, and observability, and more importantly, how it operates in a real-world setting.

RPS Quest: A hands-on MLOps proof of concept by an academic aspiring to get into industry!  Play value-based rock-paper-scissors against one of three machine learning models that are continuously trained and monitored with real data!  

🧭 Practical Infrastructure: 
* a single-node k3s Kubernetes cluster orchestrates the workloads, 
* FastAPI powers the gameplay API, 
* SQLite keeps durable state, 
* MinIO serves as the low-latency model cache,
* DagsHub/MLflow handle registry + experiment tracking,
* Docker images built from the repo are the authoritative release artifact.

🧮 Gameplay runs to 10 points with deterministic round multipliers. Players choose a model to play against:
* 🧠 Brian - feedforward neural network
* 🌲 Forrest - XGBoost
* 🪵 Logan - multinomial logistic regression

🔧 How it works: 
* Models predict player actions based as a Markov decision process (state = three action lookback).   
* Each model is trained to historic data (ingested in real time as gameplay ensues).  
* Four live aliases (Production, B, shadow1, shadow2) within model types, each with differing hyperparameter configurations. 
* Models are promoted according to fix rules (win rates, prediction accuracy)
* Players play against Production or B (50/50 split) to emulate A/B testing (monitor game win rates), and shadow models assess accuracies of potential models.

⚙️ Operations I now automated end-to-end:
- CronJobs trigger sequential training runs that respect a 4 GB RAM / 8 GB swap budget, syncing artifacts to MinIO and DagsHub.  (Hardware is light since I bought this server solely for this project!)
- A 50-feature contract (lagged moves, historical round values, score context) stays identical between training and serving; parity harnesses replay live games to catch drift.
- Grafana Cloud publish action accuracy, game win rates, training outcomes, and promotion decisions. A custom ledger endpoint feeds Grafana’s JSON datasource without custom plugins.
- Auto-promotion compares Production vs. B once each alias has 3 or more completed games
- Periodically rearranges B, shadow1, shadow2 according to accuracy

📈 If headcount were to grow, I'd make some changes:
- Infrastructure: adjust SQLite to PostgreSQL, add Kafka for event sourcing, layer Spark for feature windows and better data handling, and wrap it all in GitOps with autoscaling. 
- ML Side: Broader hyperparameter sweeps, add value-aware objectives so policies balance win probability with point preservation, stratified checks by user profile keeps promotion decisions honest as the player base grows.

🗒️ Interesting notes:
- Logistic regression noticably has the worst accuracy and win rate.  This makes sense as player behavior is highly complex and nonlinear.
- I was able to include a "danger" penalty in the loss function to help the neural network and logistic regression models; this hedges against losing to the opponent's most likely event


Try it yourself!

* Play the Game: [mlops-rps.uk/ui-lite](https://mlops-rps.uk/ui-lite)  ([debug version](https://mlops-rps.uk/ui-lite-debug))
* View Grafana Metrics: [grafana metrics](https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95)
* Read More: [jimmyrisk.github.io/rps](https://jimmyrisk.github.io/rps/)
* Github URL: [github.com/jimmyrisk/rps](https://github.com/jimmyrisk/rps)
* Dagshub URL: [dagshub.com/jimmyrisk/rps](https://dagshub.com/jimmyrisk/rps)

<iframe
  src="{{ page.api_base }}/ui-lite?api_base={{ page.api_base }}"
  title="RPS Quest Lite"
  style="width:110%;min-height:780px;border:1px solid #d0d7de;border-radius:12px;"
  loading="lazy"
  allowfullscreen
>
</iframe>

---
OLD BELOW
---

How does it work?

* Rock-Paper-Scissors with deterministic round multipliers (1.0–2.0); you win by driving the opponent from 100 → 0 points.
* Models ingest a 50-feature vector: lagged user/bot moves, prior results, historical round values, score differentials, and an easy-mode flag at index 49.
* Expected values are recomputed every round so the policy picks the move with the best outcome under current stakes; easy mode tilts the distribution toward weaker moves for accessibility.
* Gambit openings cover rounds 1–3 before ML predictions kick in, keeping the first turns approachable.

How is it MLOps?

* Kubernetes CronJobs retrain models, sync artifacts to MinIO, and log to MLflow so every alias points to a known run.
* Auto-promotion compares Production vs. B, reorders challengers by live accuracy.
* SQLite stores raw games; feature extraction, training code, and parity harnesses live in the repo so audits are reproducible.
* MinIO caches the 12 active models for sub-second loads; DagsHub keeps the full experiment history.
* MLflow pyfunc wrappers guarantee serving parity across model families.


* Initial training on some basic "human-tendency" algorithms I built (~5000 events, approx. 200 games)
* Use these to insert games on a schedule if new human data hasn't been obtained, to keep training and metrics fresh.

* Promotion rules: 
  1. After each staggered training run, compare Production vs. B once both have logged ≥3 completed games; swap if the challenger is winning.
  2. Re-rank the challenger aliases by live action accuracy so the next-best model is always queued in slot B.

How would I scale it?  
* Backend: SQLite → PostgreSQL, Kafka for event sourcing, Spark for sliding windows and clean data management, Amazon Simple Storage Service (S3) as durable artifact store (keeping MinIO as cache), add a Horizontal Pod Autoscaler (HPA) and GitOps.

