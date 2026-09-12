# factory-pulse
# FactoryPulse

**Real-Time Predictive Maintenance & Equipment Intelligence Platform for Smart Manufacturing**

FactoryPulse is an end-to-end industrial AI platform that ingests high-frequency sensor telemetry from factory machines, detects anomalies in near-real-time, forecasts Remaining Useful Life (RUL), and exposes an agentic maintenance copilot that diagnoses faults by combining live model outputs with a RAG knowledge base built from equipment manuals, maintenance logs, and fault-code documentation.

Built entirely on public datasets (NASA C-MAPSS, NASA IMS bearings, AI4I 2020), replayed through a simulator to emulate live machines — no factory access required.

---

## 🚀 Why This Exists

Unplanned equipment downtime costs manufacturers an estimated **$50B annually**. Most plants run either:
- **Reactive maintenance** — fix after failure, causing catastrophic breakdowns and safety incidents
- **Calendar-based maintenance** — replace parts on a fixed schedule, wasting healthy components while still missing developing faults

FactoryPulse instead **learns each machine's normal behavior**, **predicts failures before they happen**, and turns raw anomalies into **diagnosed, explained, prioritized work orders** via an LLM copilot grounded in the plant's own documentation.

---

## ✨ Key Features

- 📡 **Real-time ingestion** — streaming sensor telemetry via MQTT/HTTP into TimescaleDB
- 🔍 **Anomaly detection** — LSTM autoencoder + Isolation Forest ensemble with per-machine adaptive thresholds
- ⏳ **RUL forecasting** — Transformer-based Remaining Useful Life prediction (RMSE ≤ 20 cycles on C-MAPSS FD001)
- 🌡️ **Thermal imaging** — OpenCV-based hotspot detection fused into machine health scoring
- 🤖 **GenAI Maintenance Copilot** — LangGraph agent with hybrid RAG (dense + BM25 + reranking) and a groundedness gate to prevent hallucinated claims
- 🔌 **MCP interface** — copilot tools exposed via Model Context Protocol for external LLM clients
- 📊 **Ops console** — Streamlit dashboard for fleet health, alerts, live charts, and copilot chat
- 🔁 **Full MLOps loop** — MLflow experiment tracking, DVC data versioning, shadow deployment, drift detection

---

## 🏗️ Architecture

```
Sensor Simulator ──MQTT/HTTP──► Ingestion Service ──► TimescaleDB (raw telemetry)
(replay of C-MAPSS/IMS)              │ validation           │
                                      └─► dead-letter        │ continuous aggregates
                                                              ▼
                                        Feature Pipeline (batch + streaming)
                                            │ rolling stats, FFT bands, trends
                                            ├─► feature tables (TimescaleDB)
                                            └─► Redis (online feature cache)
                                                              │
MLflow ◄── Training Pipelines (PyTorch / LightGBM / XGBoost) ◄── DVC data
registry │                                                    │
         │                                                    ▼
         └────► Inference Service (FastAPI) ──► Alert Engine ──► PostgreSQL (alerts)
                    │ anomaly + RUL + failure mode        │ webhooks
                    ▼                                     ▼
              Copilot Service (FastAPI + LangGraph) ◄──── Qdrant (manuals, logs)
                    │ tools: health, RUL, manuals, history
                    ▼
              Streamlit Console (fleet, alerts, chat, drift)
```

Every service is independently containerized, health-checked, and covered by CI/CD.

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| Streaming / Ingestion | MQTT/HTTP, FastAPI |
| Time-series storage | TimescaleDB |
| Caching | Redis |
| ML Training | PyTorch, LightGBM, XGBoost |
| Experiment Tracking | MLflow |
| Data Versioning | DVC |
| Vector Search | Qdrant (hybrid dense + BM25) |
| Agent Framework | LangGraph |
| Embeddings / Reranking | bge-small-en-v1.5, bge-reranker-base |
| Frontend | Streamlit |
| Relational DB | PostgreSQL, Alembic migrations |
| Auth | JWT, bcrypt |
| Monitoring | Prometheus, loguru (structured JSON logs) |
| CI/CD | GitHub Actions, Docker Compose |

---

## 📁 Project Structure

```
factorypulse/
├── services/
│   ├── simulator/       # replays public datasets as live telemetry
│   ├── ingestion/        # validates & writes telemetry to TimescaleDB
│   ├── inference/         # serves anomaly + RUL + failure-mode predictions
│   ├── copilot/            # LangGraph agent + RAG-based diagnostic copilot
│   └── console/             # Streamlit operations dashboard
├── ml/                        # shared training package
│   ├── data/                   # dataset loaders & canonical schema
│   ├── features/                 # rolling stats, spectral features, trends
│   ├── models/                     # autoencoder, RUL transformer, classifiers
│   ├── training/                     # DVC training stages
│   ├── thermal/                        # OpenCV hotspot detection
│   └── monitoring/                       # drift detection jobs
├── db/                                     # migrations & seed scripts
├── tests/                                    # unit, integration, API, load, e2e
├── docs/                                       # architecture docs & ADRs
├── docker-compose.yml
└── dvc.yaml
```

---

## 📈 Business Objectives

- ✅ Detect developing faults **≥ 48 hours** before functional failure
- ✅ RUL prediction with **RMSE ≤ 20 cycles** on C-MAPSS FD001
- ✅ Reduce false alarms by **≥ 60%** vs. static-threshold baseline
- ✅ End-to-end alert latency **≤ 5 seconds**
- ✅ Copilot answers with **≥ 90%** citation-supported claims
- ✅ Cut simulated maintenance cost by **≥ 25%** vs. calendar-based policy

---

## 🖥️ Getting Started

```bash
# Clone the repo
git clone https://github.com/<your-username>/factorypulse.git
cd factorypulse

# Spin up the full stack (TimescaleDB, Redis, Qdrant, MLflow, all services)
make up

# Seed demo machines, sensors, and documentation
make seed

# Open the console
# http://localhost:8501
```

> One command gives you a live demo with a machine visibly failing and a copilot ready to diagnose it.

---

## 🔬 Example Workflow

> **02:14, Night Shift** — Press CNC-07's vibration RMS starts drifting. The autoencoder's reconstruction error crosses its adaptive threshold; RUL drops to 62 hours. The alert engine opens a WARN that escalates to CRITICAL by 05:00.
>
> Technician opens the alert — the copilot has already pulled the live health trend, matched the spectral signature to outer-race bearing wear, cited the manual, and drafted a work order with parts and safety steps for review.

---

## 🧪 Testing

- **Unit** — validators, feature transforms, alert hysteresis logic
- **Integration** — ingestion → TimescaleDB, retriever → Qdrant (via testcontainers)
- **API** — contract tests per service with role-based auth checks
- **AI Evaluation** — golden-set copilot eval (retrieval hit-rate, groundedness, citation precision)
- **Load** — Locust profiles for ingestion & inference throughput
- **E2E** — full pipeline: simulator → alert → grounded work order

---

## 🗺️ Roadmap

- Kafka-based ingestion for horizontal scale
- Federated per-plant models
- Edge inference (ONNX on Raspberry Pi gateways)
- RL-based maintenance scheduling
- CMMS integration (e.g., SAP PM)
- Multimodal copilot (technician-submitted photos)
- Kubernetes + Helm deployment

---

## 📚 Key Concepts

| Term | Meaning |
|---|---|
| **RUL** | Remaining Useful Life — predicted operating time before failure |
| **Autoencoder anomaly detection** | Flags machines that no longer reconstruct like their "healthy" baseline |
| **Hysteresis alerting** | Alert opens above a high threshold, closes only below a lower one — prevents flapping |
| **Shadow deployment** | A candidate model scores live traffic silently for safe evaluation before promotion |
| **Groundedness gate** | Every copilot-generated claim is checked against retrieved evidence before being shown |
| **MCP** | Model Context Protocol — lets external LLM clients call the copilot's tools |

---

## 📄 License

MIT (or update to your preferred license)

---

## 🙋 About

Built as a portfolio project demonstrating end-to-end industrial AI systems design — streaming data engineering, applied ML/deep learning, agentic GenAI with retrieval and hallucination control, and production-grade MLOps.
