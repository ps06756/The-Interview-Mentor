# ML System Design -- Problem Bank, Walkthroughs & Calculations

---

## Problem 1: Recommendation System for E-Commerce

### Requirements & Back-of-the-Envelope
- **Scale**: 50M DAU, 500M products, 2B interaction events per day.
- **Latency**: Candidate retrieval < 20ms, full ranking < 100ms end-to-end.
- **Throughput**: 50M users * avg 10 sessions/day = 500M recommendation requests/day (~6,000 QPS).
- **Data**: User click/purchase logs, product metadata, user profiles.

### Architecture Walkthrough

**1. Candidate Generation**
- Two-tower model: user encoder and item encoder trained on interaction data.
- Pre-compute item embeddings nightly. Store in FAISS/ScaNN ANN index.
- At serving time, compute user embedding from recent interactions, query ANN for top 500 candidates.
- Also include rule-based candidates: trending items, recently viewed, same-category popular items.

**2. Feature Store**
- Offline features (updated daily via Spark): user lifetime purchase count, category affinities, avg order value.
- Near-real-time features (updated via Flink, seconds-fresh): items viewed in current session, cart contents, last click category.
- Online store: Redis cluster with feature vectors keyed by user_id and item_id.
- Offline store: Hive/S3 partitioned by date for training data generation.

**3. Ranking Model**
- Deep ranking model (Wide & Deep or DCN-v2) trained on click and purchase labels.
- Input: user features (from feature store), item features (from feature store), cross features, context (time of day, device).
- Served via TF Serving with batched inference. GPU instances for throughput.

**4. A/B Testing**
- Traffic split via experiment framework (feature flags by user_id hash).
- Primary metric: revenue per session. Secondary: CTR, conversion rate.
- Guardrail metrics: page load time, session duration (ensure recommendations do not degrade UX).
- Run for 2+ weeks to account for novelty effect and day-of-week patterns.

**5. Monitoring & Data Flywheel**
- Track prediction score distributions daily. Alert on KL-divergence > threshold.
- Log all impressions, clicks, and purchases to Kafka for retraining data.
- Daily retraining pipeline with fresh interaction data. Weekly full retrain with hyperparameter search.
- Shadow scoring: new model scores live traffic in parallel before promotion.

---

## Problem 2: Fraud Detection System

### Requirements & Back-of-the-Envelope
- **Scale**: 10,000 transactions per second peak, 500M transactions per day.
- **Latency**: Decision within 100ms (blocking the payment flow).
- **Labels**: ~0.1% fraud rate. Labels arrive with delay (chargebacks take 30-90 days).
- **Cost**: False negatives are expensive (fraud loss). False positives damage user experience.

### Architecture Walkthrough

**1. Rules Engine (First Pass)**
- Hard rules: blocked countries, known fraudulent device fingerprints, velocity limits.
- Runs in < 5ms. Catches ~30% of fraud with very high precision.
- Easy to update without model retraining.

**2. Feature Engineering**
- Real-time aggregates via Flink: transaction count in last 1h/24h/7d per user, per card, per device.
- Static features: account age, verified status, historical chargeback rate.
- Graph features: connection to known fraud rings via shared devices/addresses.
- All served from Redis with < 5ms lookup.

**3. ML Model**
- XGBoost or LightGBM for primary scoring (fast inference, handles tabular data well).
- Handle class imbalance: use focal loss or SMOTE during training. Evaluate on precision-recall curves, not accuracy.
- Calibrated probabilities for threshold-based decisions.

**4. Decision Logic**
- Score > 0.9: auto-block, notify user.
- Score 0.7-0.9: hold for human review (SLA: 30 minutes).
- Score < 0.7: approve.
- Thresholds tuned based on cost-benefit analysis (fraud loss vs friction cost).

**5. Feedback Loop & Monitoring**
- Human review labels fed back within hours. Chargeback labels arrive in 30-90 days.
- Monitor: false positive rate daily, fraud loss weekly.
- Concept drift detection: fraud patterns shift (e.g., new attack vector). Track feature importance stability and prediction distribution.
- Retrain monthly with fresh labels. Emergency retrain if fraud spike detected.

---

## Problem 3: Model Serving Platform

### Requirements & Back-of-the-Envelope
- **Scale**: 20 teams, 100+ models in production, 500K total QPS across all models.
- **Frameworks**: TensorFlow, PyTorch, XGBoost, ONNX.
- **Latency**: Varies by model. Some need < 10ms (search ranking), others tolerate 500ms (batch recommendations).
- **Availability**: 99.99% uptime SLO for critical models.

### Architecture Walkthrough

**1. Model Registry**
- MLflow for artifact storage (S3-backed), metadata tracking, lineage.
- CI/CD pipeline: model passes validation (schema check, performance regression test on holdout set) before registration.
- Immutable model versions. Rollback by repointing serving to previous version.

**2. Serving Runtime**
- NVIDIA Triton Inference Server: supports TF, PyTorch, ONNX, tree models in one runtime.
- Models packaged as Docker images with config (max batch size, instance count, GPU/CPU).
- Request batching for throughput optimization on GPU models.

**3. Infrastructure**
- Kubernetes with separate node pools: GPU (A10G) for deep learning, CPU (c5.4xlarge) for tree models.
- HPA based on custom metrics: p99 latency, request queue depth, GPU utilization.
- Pre-warm instances for latency-sensitive models to avoid cold starts.

**4. Traffic Management**
- Istio service mesh for traffic splitting (canary deployments).
- Canary rollout: 1% -> 10% -> 50% -> 100% with automated rollback on SLO violation.
- A/B test routing: experiment framework assigns users to model versions via consistent hashing.

**5. Monitoring & Observability**
- Prometheus metrics: latency (p50, p95, p99), throughput, error rate, GPU/CPU utilization per model.
- Grafana dashboards per team and per model.
- Input feature drift detection: KL divergence on feature distributions, alert if > threshold.
- Prediction drift: track score distribution. Alert on significant shift.
- Cost attribution: track compute spend per model per team.

---

## Problem 4: Search Ranking System

### Requirements & Back-of-the-Envelope
- **Scale**: 100M queries per day, 10B documents in index.
- **Latency**: End-to-end < 200ms (retrieval + ranking).
- **Relevance**: Measured by NDCG@10, click-through rate, session success rate.

### Architecture Walkthrough

**1. Retrieval Stage**
- Inverted index (Elasticsearch/Lucene) for text matching: BM25 scoring.
- Semantic retrieval: bi-encoder model encodes queries and documents into embeddings. ANN search via FAISS.
- Combine: union of BM25 top-500 and semantic top-500 candidates.

**2. Ranking Stage**
- Cross-encoder or GBDT model scores each candidate.
- Features: BM25 score, semantic similarity, document quality signals, user personalization features, query-document interaction features.
- Distill a large cross-encoder into a smaller model for serving latency.

**3. Online Learning**
- Log search results and clicks. Use click models (position-debiased) to generate training labels.
- Daily model updates with fresh click data. Weekly full retraining.

**4. Evaluation**
- Offline: NDCG, MAP on human-rated query-document pairs.
- Online: A/B test measuring CTR, session success rate, query reformulation rate (lower is better).
- Interleaving experiments for faster signal than A/B tests.

---

## Problem 5: Real-Time Personalization Engine

### Requirements & Back-of-the-Envelope
- **Scale**: 100M users, 1B events per day.
- **Latency**: Personalized content within 50ms of page load.
- **Context**: Must adapt within a single session (e.g., user starts browsing electronics, switch to fashion).

### Architecture Walkthrough

**1. Session-Aware Features**
- Stream processing (Flink) maintains per-user session state: last N clicks, dwell time, category distribution.
- Features written to Redis with TTL matching session timeout.

**2. Multi-Armed Bandit for Exploration**
- Thompson Sampling or contextual bandit for content slot allocation.
- Balances exploitation (show what we know works) with exploration (try new content to learn preferences).
- Reward signal: click, add-to-cart, purchase (with different weights).

**3. Model Architecture**
- Lightweight model (logistic regression or small neural net) for real-time scoring.
- User embedding updated incrementally with each new interaction (online learning or cached session embedding).
- Pre-computed item embeddings refreshed daily.

**4. Cold Start Handling**
- New users: use contextual signals (device, location, referrer, time of day) to select initial content.
- New items: use content-based features (category, title embedding, image embedding) until sufficient interaction data.
- Gradually blend collaborative filtering signals as data accumulates.

**5. Data Flywheel**
- Every impression, click, and conversion logged.
- Bandit reward signals update exploration policy in near-real-time.
- Weekly offline analysis: which content categories drive the most engagement? Feed back into editorial and merchandising teams.
