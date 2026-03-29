# Autonomous Crypto Swing-Trading Agent

An event-driven, AI-powered crypto swing trading system built on Kubernetes.  
This project showcases **Platform Engineering**, **GitOps**, and **Agentic AI** by combining quantitative market data with qualitative sentiment analysis via a RAG (Retrieval-Augmented Generation) pipeline.

> **💡 Why this architecture?** > While a daily swing-trading script could technically run as a simple cron job, this project is deliberately engineered on Kubernetes to showcase scalable Platform Engineering, distributed systems, and modern AI orchestration patterns.

---

## Overview

Traditional crypto bots rely only on technical indicators and often fail during macro shifts. Pure LLM-based bots lack precision, context, and risk management.

This project introduces a **Dual-Brain Architecture**:

- **Quantitative Engine** → Processes OHLCV data & technical indicators  
- **Agentic LLM (Gemini + LangGraph)** → Analyzes news & sentiment using Vector Search (RAG)  

Together, they generate daily **swing trade decisions**, complete with memory of past trades and robust observability.

---

## Architecture



The system runs as a **daily event-driven pipeline** orchestrated via Argo Workflows.

### Infrastructure & GitOps
ArgoCD → Helm Charts → TimescaleDB (`pgvector`) + Prometheus/Grafana

### Phase 1: Data Ingestion & Embedding
Argo Cron
├── Market Data Fetcher → TimescaleDB
└── Sentiment Scraper → Text Embeddings → `pgvector`

### Phase 2: Agentic Intelligence (Stateful)
`pgvector` (RAG) ↔ LangGraph Agent (w/ Checkpointer Memory) ↔ Gemini API
↓
Trade Signals DB

### Phase 3: Execution & Observability
Trade Signals → Python Execution Engine (`ccxt`)
├── Risk Management & Position Sizing
├── Exchange API Execution (Supports `PAPER_TRADE=true`)
└── LangSmith (LLM Tracing) + Grafana (System Metrics)

---

## Core Components

### 1. Data Layer
- **TimescaleDB** (PostgreSQL extension) for efficient time-series OHLCV data.
- **`pgvector`** for storing semantic embeddings of financial news and sentiment data.

### 2. Intelligence Gatherer (RAG Pipeline)
- Scrapes financial news and the Fear & Greed Index.
- Converts unstructured news into vector embeddings, allowing the agent to pull only the most relevant macro context for the day's decision.

### 3. Strategist (Agentic AI)
- Powered by **Gemini API** and **LangGraph**.
- **Stateful Memory:** Uses LangGraph's checkpointer backed by Postgres so the agent "remembers" why it entered a position days ago.
- Evaluated and debugged via **LangSmith** to trace token usage and agent reasoning steps.

### 4. Execution Engine
- Written in **Python** utilizing the `ccxt` library for universal exchange compatibility.
- Enforces strict risk management (stop-loss, max position size).
- Includes a **Dry-Run Mode** (`PAPER_TRADE=true`) for safe forward-testing.

---

## Tech Stack

| Category              | Tools |
|----------------------|------|
| **Deployment & GitOps** | ArgoCD, Helm |
| **Orchestration** | Kubernetes (kind), Argo Workflows |
| **Database** | PostgreSQL, TimescaleDB, `pgvector` |
| **AI / Agent** | Gemini API, LangGraph, LangSmith |
| **Execution** | Python, `ccxt` |
| **Observability** | Prometheus, Grafana |

---

## ⚡ Getting Started

### Prerequisites

- Docker / Docker Desktop
- kind & kubectl
- Helm
- Gemini API Key & LangSmith API Key

---

### 1. Create Kubernetes Cluster

```bash
kind create cluster --name crypto-agent
```

### 2. Install ArgoCD & Workflows
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create namespace argo
kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/latest/download/install.yaml
```

### 3. Deploy Infrastructure (Database & Observability)
```bash
helm repo add timescale https://charts.timescale.com
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Postgres with Timescale & pgvector enabled
helm install crypto-db timescale/timescaledb -n default

# Install Kube-Prometheus-Stack for Grafana/Monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n default
```

### 4. Configure Environment Variables
Create a `.env` file:

```ini
EXCHANGE_API_KEY=your_key_here
EXCHANGE_SECRET=your_secret_here
PAPER_TRADE=true
GEMINI_API_KEY=your_gemini_api_key
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_key
DATABASE_URL=postgres://user:pass@crypto-db:5432/cryptodata
```

### 5. Run Trading Pipeline
```bash
argo submit -n argo k8s/argo-trading-workflow.yaml --watch
```

---

## Roadmap & Evaluation

- [x] Paper trading and stateful agent memory
- [x] Semantic search (RAG) for news context
- [ ] Historical backtesting engine integration
- [ ] Multi-agent debate (e.g., Bull Agent vs. Bear Agent) before final execution

## Disclaimer
This project is for educational and engineering purposes only.

- **Not financial advice.**
- Do not use with real funds without proper backtesting and risk management. 
- Always default to `PAPER_TRADE=true` during testing.

## Contributing
Contributions are welcome! Feel free to open issues or submit pull requests.

## License
MIT License
