As an academic aspiring to get into industry, I realized I had to beef up my skills beyond academic research and kaggle datasets. What better way than to build a MLOps enterprise from scratch?  

In three weeks I created RPS Quest - a full-stack MLOps proof of concept that helped me own the complete lifecycle: modeling, infrastructure, deployment, and observability. Fully open source with live dashboards.


Try it: https://mlops-rps.uk/ui-lite

🧮 The System:
Players choose a model to play against:
* 🧠 Brian - feedforward neural network
* 🌲 Forrest - XGBoost
* 🪵 Logan - multinomial logistic regression

Treated as a Markov decision process (state = three action lookback). Real-time training on live player data with automated A/B testing. Four aliases per model (Production, B, shadow1, shadow2) with auto-promotion based on win rates and accuracy.



🧭 Infrastructure: 
* k3s Kubernetes cluster orchestrating Python/FastAPI backend
* PyTorch + XGBoost for modeling
* MLflow/DagsHub for registry + tracking
* MinIO for model caching
* Dockerized for production parity



⚙️ Key MLOps Features:
* CronJob-triggered training pipelines syncing to MinIO/DagsHub
* 50-feature contract maintained across training/serving
* Grafana dashboards monitoring accuracy, win rates, and auto-promotions
* Parity testing using live game replays to catch drift



📈 Production Insights:
* Custom "danger penalty" loss function boosted neural net win rates
* XGBoost provides interpretable feature importance on player tendencies
* Models improve as real player data accumulates  



🔗 Links:
* Play: https://mlops-rps.uk/ui-lite
* Metrics: https://jimmyrisk41.grafana.net/public-dashboards/786f7f916d084387b726ac4e7e8a7d95
* Details: https://jimmyrisk.github.io/rps/
* GitHub: https://github.com/jimmyrisk/rps
* DagsHub: https://dagshub.com/jimmyrisk/rps
