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

<iframe
  src="{{ page.api_base }}/ui-lite?api_base={{ page.api_base }}"
  title="RPS Quest Lite"
  style="width:110%;min-height:780px;border:1px solid #d0d7de;border-radius:12px;"
  loading="lazy"
  allowfullscreen
>
</iframe>

RPS Quest: A MLOps proof of concept by an academic aspiring to get into industry!

Looking at job ads, I realized I had to beef up my skills beyond the academic research and kaggle datasets.  What better way than to build a MLOps enterprise from scratch?

(TODO: MERGE 1 and 2.  MAKE IT "LOOK GOOD" TO EMPLOYERS THAT I UNDERSTAND MLOPS, DO NOT LIE.  THE IMPROVEMENTS SECTION COME AFTER)
1. Original research and intent: A compact but “enterprise-shaped” stack, with Kubernetes on Amazon Web Services (AWS) Elastic Compute Cloud (EC2), Kafka for streams, PostgreSQL for storage, Apache Spark for compute, FastAPI at the edge—built/pushed via Elastic Container Registry (ECR).
2. The product: A lean, production-aware proof: k3s, one app pod, SQLite for simplicity, MinIO for sub-second cold loads, DagsHub/MLflow for auditability, and scheduled Jobs for training. The goal was reliability under tight memory, not heroics.  Fully open source (backend, frontend, server configurations, metrics monitoring).

Try it yourself!

* Play the Game: [mlops-rps.uk/ui-lite](https://mlops-rps.uk/ui-lite)  ([debug version](mlops-rps.uk/ui-lite-debug))
* View Grafana Metrics: [grafana metrics](https://jimmyrisk41.grafana.net/public-dashboards/4da302b832f04242be45745da876cc54)
* Read More: [jimmyrisk.github.io/rps](https://jimmyrisk.github.io/rps/)
* Github URL: [github.com/jimmyrisk/rps](https://github.com/jimmyrisk/rps)
* Dagshub URL: [dagshub.com/jimmyrisk/rps](https://dagshub.com/jimmyrisk/rps)



How does it work?

* Rock-Paper-Scissors with random values; reduce opponent's life to 0 from 100.
* Three ML model opponents:
  * 🧠 Brian (feedforward neural network)
  * 🌲 Forrest (XGBoost; random forest)
  * 🪵 Logan (Logsistic regression)
* Each uses a 3-event lag feature set including [fill in] (first three choices are according to a [RPS gambit](https://wrpsa.com/gambits-of-rock-paper-scissors/))
* Predict the probabilities of your move, followed by a fast expected value calculation (simplified HJB equation) to determine optimal move.

How is it MLOps?

(need to mention the terms here.  cronjob, kubernetes.  also something about SQL, more about mlflow?)
* Each model has four versions ("Production", "B", "Shadow1", "Shadow2") that evaluate a test accuracy for every event the opponent picks; monitored in real time via kubernetes/grafana/prometheus
* "Production" and "B" are chosen in actual games (50/50 split); win rates recorded in real time
* Automatic promotion on a schedule (see below)
* Models train on a schedule
* Runs on a single pod/server I bought for this project.  Hardware limitations keeps the 12 aliased models on the server with minio, all models stored in dagshub
* All training and prediction unified via mlflow across model classes
* Custom loss functions to hedge against opponent thinking one step ahead


* Initial training on some basic "human-tendency" algorithms I built (~5000 events, approx. 200 games)
* Use these to insert games on a schedule if new human data hasn't been obtained, to keep training and metrics fresh.

* Promotion rules: 
  1. Check periodically between "Production" and "B", perform a two-sample test for proportions (num. wins and losses over 5 for each); promote if B is better.  *(For POC, simplify to the winning model after 3 games.)*
  2. Rearrange "B", "Shadow1", "Shadow2" according to their highest prediction accuracy.

How would I scale it?  
* Backend: SQLite → PostgreSQL, Kafka for event sourcing, Spark for sliding windows and clean data management, Amazon Simple Storage Service (S3) as durable artifact store (keeping MinIO as cache), add a Horizontal Pod Autoscaler (HPA) and GitOps.
* ML Side: More comprehensive hyperparameter tuning, fully retiring shadow models that have been underperforming and introduce new ones, adjusted loss functions to optimize value in addition to likelihood.  Adjust promotion, training, and testing to be stratified based on users (or set guardrails to ensure sufficient data diversity before doing checks)
