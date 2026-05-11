# NEON-SYX
### A fully local multi-agent simulation engine for modeling public opinion and social dynamics

> Built on the architectural foundations of MiroFish, reengineered for pre-launch product intelligence.

---

## What is NEON-SYX?

NEON-SYX is a **fully local, privacy-preserving multi-agent simulation platform** that predicts how the public will react to a product — *before it launches*.

Instead of running surveys or focus groups after a product is built, NEON-SYX lets you upload a product document (pitch deck, spec sheet, launch brief) and simulates hundreds of AI agents — each with distinct personalities, biases, and social behaviours — debating the product on a synthetic social media platform.

The result: structured sentiment reports, influence maps, and stakeholder reaction models, all generated in under 4 hours, all on local infrastructure with no data leaving your environment.

---

## Key Features

- **Zero cloud dependency** — all processing runs locally via Ollama + Neo4j + Docker
- **Document-agnostic** — accepts PDF, Markdown, or plain text product specifications
- **Hundreds of AI agents** — each with OCEAN personality traits, psychographic profiles, and graph-backed memory
- **Knowledge graph memory** — Neo4j stores entities, relationships, and agent interactions with vector + BM25 hybrid retrieval
- **5-stage deterministic pipeline** — from document ingestion to interactive agent Q&A
- **Privacy-preserving** — sensitive product documents never leave your controlled environment
- **Kaggle GPU proxy** — run production-quality LLMs on Kaggle's T4 GPUs via ngrok tunnel, no local GPU required

---

## Five-Stage Pipeline

```
Stage 1 — Graph Build       Upload document → extract ontology → build Neo4j knowledge graph
Stage 2 — Env Setup         Generate agent personas from graph entities
Stage 3 — Simulation        Run multi-agent social media simulation (Reddit-style)
Stage 4 — Report            Generate structured sentiment analysis + risk clusters
Stage 5 — Interaction       Chat directly with agents using full graph memory
```

---

## Technology Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | Vue.js / Vite | 3.5.25 / 7.2.7 | Reactive SPA with D3.js graph visualisation |
| Backend | Flask + Python | Latest / 3.11+ | REST API, pipeline orchestration |
| Graph DB | Neo4j CE | 5.15 | Knowledge graph, vector + full-text index |
| LLM Runtime | Ollama + qwen2.5 | 14b | Agent generation, NER/RE, report synthesis |
| Embeddings | nomic-embed-text | Latest | 768-dimensional vector embeddings |
| Deployment | Docker Compose | Latest | Multi-container orchestration |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────┐
│                   Frontend                       │
│         Vue.js SPA (port 3000)                  │
│    Home.vue → Process.vue → Step1–5.vue         │
└────────────────────┬────────────────────────────┘
                     │ HTTP
┌────────────────────▼────────────────────────────┐
│                   Backend                        │
│          Flask REST API (port 5001)             │
│   /api/graph   /api/simulation   /api/report    │
└──────────┬──────────────────┬───────────────────┘
           │                  │
┌──────────▼──────┐  ┌────────▼────────────────────┐
│   Neo4j CE 5.15 │  │   Ollama (via ngrok tunnel) │
│  port 7687/7474 │  │   qwen2.5:14b + nomic-embed │
│  Knowledge Graph│  │   Running on Kaggle T4 GPUs │
└─────────────────┘  └─────────────────────────────┘
```

---

## Prerequisites

- Docker Desktop
- Python 3.11+
- Node.js 18+
- A [Kaggle](https://kaggle.com) account with GPU access
- A free [ngrok](https://dashboard.ngrok.com) account + auth token

---

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/neon-syx.git
cd neon-syx
```

### 2. Set up the Kaggle Ollama server

The LLM runs on Kaggle's free T4 GPUs. You need to set this up once and re-run it each session.

**First time only — save the model to a Kaggle dataset:**

Run `Neon-Syx-save-14b-model-dataset.ipynb` on Kaggle to download and save `qwen2.5:14b` and `nomic-embed-text` to your personal Kaggle dataset. This takes ~20 minutes.

**Every session — start the Kaggle server:**

Open the main Kaggle notebook `Neon-Syx-kaggle-model.ipynb` and run all cells in order:

```
Cell 1 — Check GPU availability
Cell 2 — Install Ollama engine
Cell 3 — Restore models from your saved dataset
Cell 4 — Start Ollama server
Cell 5 — Smoke test (verify models load correctly)
Cell 6 — Start ngrok tunnel → copy the output URL
Cell 7 — Keep-alive loop (leave running)
```

Copy the printed `.env` values from Cell 6.

### 3. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` with the ngrok URL from your Kaggle notebook:

```env
# LLM — Kaggle Ollama via ngrok tunnel
LLM_API_KEY=ollama
LLM_BASE_URL=https://YOUR-NGROK-URL.ngrok-free.app/v1
LLM_MODEL_NAME=qwen2.5:14b

# Neo4j — local Docker
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=mirofish

# Embeddings — also via Kaggle tunnel
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_BASE_URL=https://YOUR-NGROK-URL.ngrok-free.app

# Optional: increase for large documents
OLLAMA_NUM_CTX=16384
LLM_TIMEOUT=600
```

### 4. Start Neo4j via Docker

```bash
docker run -d \
  --name neo4j-neon \
  --env NEO4J_AUTH=neo4j/mirofish \
  --env NEO4J_server_memory_heap_max__size=1g \
  -p 7474:7474 -p 7687:7687 \
  neo4j:5.15
```

Or using Docker Compose:

```bash
docker compose up -d neo4j
```

### 5. Start the backend

```bash
cd backend
pip install -r requirements.txt
python run.py
```

Backend runs at `http://localhost:5001`

### 6. Start the frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend runs at `http://localhost:3000`

---

## Running a Simulation

1. Open `http://localhost:3000`
2. Upload a product document (PDF, Markdown, or plain text)
3. Enter a simulation requirement (e.g. *"Simulate public reaction to this product launch on social media"*)
4. Click through the 5 stages:
   - **Stage 1** — Watch the knowledge graph build in real time (D3.js visualisation)
   - **Stage 2** — Review and configure generated agent personas
   - **Stage 3** — Run the simulation (72 rounds ≈ ~2 hours)
   - **Stage 4** — Read the structured sentiment report
   - **Stage 5** — Chat directly with individual agents

---

## NLP Techniques Used

**Named Entity Recognition (NER)** — Zero-shot and few-shot extraction of domain-specific entities (ProductFeature, TargetAudience, MarketPainPoint, Competitor) via local LLM calls.

**Relation Extraction (RE)** — Triplet-level extraction of directional semantic relationships (e.g. Feature A → SOLVES → Pain Point B), stored as typed Neo4j edges.

**Hybrid RAG** — Retrieval-Augmented Generation using weighted fusion of `0.7 × vector similarity + 0.3 × BM25` keyword score for agent memory queries.

**Agentic Prompt Engineering** — Each agent receives a structured system prompt containing OCEAN personality traits, socioeconomic bias, domain scepticism level, and graph-derived interests — all extracted from the input document.

**Temporal Sentiment Analysis** — Agent interactions (reply, upvote, downvote) are logged as time-stamped weighted signals and committed to Neo4j as temporal edges, enabling chronological sentiment tracking across the simulation clock.

---

## Kaggle Proxy Architecture

NEON-SYX uses Kaggle's free T4 GPU infrastructure to run the LLM, solving the local VRAM constraint without requiring a dedicated GPU machine.

```
Local (backend + Neo4j)
        │
        │ HTTPS via ngrok tunnel
        ▼
Kaggle Notebook (Ollama + qwen2.5:14b)
        │
        └── T4 GPU x2 (30GB total VRAM)
```

Because `LLM_BASE_URL` abstracts the endpoint entirely, switching between local Ollama and the Kaggle proxy requires only a one-line `.env` change — no code modifications.

**⚠️ Important:** The ngrok URL may change every time you restart the Kaggle notebook. Update `.env` and restart the Flask backend when this happens.

---

## Experimental Results

Validated against the **Airbnb original pitch deck** as a historical benchmark (known real-world reception, documented investor objections).

| Metric | Result |
|---|---|
| Agents generated | 34 (exact match to expected) |
| Reality seed topics | 73 |
| Simulation rounds | 72 (72-hour horizon) |
| Total pipeline runtime | ~4 hours |
| Risk clusters identified | 5 (regulatory, competition, customer scepticism, management alignment, technology) |

The simulation organically surfaced regulatory exposure and trust deficit as leading concerns — aligning with Airbnb's documented early-market friction points — without any explicit prompting toward these categories.

---

## Project Structure

```
neon-syx/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── graph.py          # Stage 1: document upload, ontology, graph build
│   │   │   ├── simulation.py     # Stages 2-3: agent generation, simulation control
│   │   │   └── report.py         # Stage 4-5: report generation, agent chat
│   │   ├── services/
│   │   │   ├── ontology_generator.py     # LLM-based ontology generation
│   │   │   ├── graph_builder.py          # NER/RE pipeline, Neo4j population
│   │   │   ├── oasis_profile_generator.py # Agent persona construction
│   │   │   ├── simulation_runner.py      # OASIS simulation execution
│   │   │   ├── report_agent.py           # Map-reduce report synthesis
│   │   │   └── graph_memory_updater.py   # Real-time Neo4j interaction logging
│   │   ├── storage/
│   │   │   ├── graph_storage.py          # Abstract GraphStorage interface
│   │   │   ├── neo4j_storage.py          # Neo4j concrete implementation
│   │   │   ├── embedding_service.py      # nomic-embed-text wrapper
│   │   │   ├── ner_extractor.py          # Named entity extraction
│   │   │   └── search_service.py         # Hybrid vector + BM25 retrieval
│   │   ├── utils/
│   │   │   └── llm_client.py             # OpenAI-compatible LLM wrapper
│   │   └── config.py                     # Environment configuration
│   └── scripts/
│       ├── file_parser.py                # PDF/Markdown ingestion + OCR fallback
│       └── run_reddit_simulation.py      # OASIS simulation entry point
├── frontend/
│   └── src/
│       ├── views/
│       │   ├── Home.vue                  # Document upload + prompt entry
│       │   └── Process.vue              # 5-stage workflow orchestrator
│       └── components/
│           ├── Step1.vue – Step5.vue    # Stage-specific UI components
│           └── HistoryDatabase.vue      # Session persistence + resume
├── Kaggle notebooks/
│   ├── Neon-Syx-kaggle-model.ipynb            # Main Kaggle server notebook
│   └── Neon-Syx-save-14b-model-dataset.ipynb  # One-time model save notebook
├── docker-compose.yml
└── README.md
```

---

## Troubleshooting

**Ontology generation times out**
- Verify the ngrok tunnel is active: `curl https://YOUR-URL.ngrok-free.app/api/tags`
- Check the model is loaded on Kaggle: run `!ollama ps` in a new cell
- Ensure `LLM_TIMEOUT=600` is set in `.env`

**ngrok tunnel goes offline**
- Re-run the ngrok cell in your Kaggle notebook
- Update `LLM_BASE_URL` and `EMBEDDING_BASE_URL` in `.env`
- Restart the Flask backend: `python run.py`

**Model unloads from GPU after inactivity**
- The keep-alive cell in the Kaggle notebook pings the model every 20 seconds.
- Make sure that cell is still running (it runs an infinite loop — keep the tab and session alive)

**Neo4j vector index warning on startup**
```
Vector indexes are not available on relationships
```
This is a known Neo4j CE limitation — relationship embeddings aren't indexed but this does not affect core functionality. The warning can be safely ignored.

---

## Acknowledgements

- [MiroFish](https://github.com/mirofish) — original open-source multi-agent simulation engine this project forks from
- [OASIS (CAMEL-AI)](https://github.com/camel-ai/oasis) — agent execution runtime
- [Ollama](https://ollama.com) — local LLM inference server
- [Neo4j Community Edition](https://neo4j.com) — graph database
- [Qwen2.5](https://huggingface.co/Qwen) — language model family
- [nomic-embed-text](https://www.nomic.ai) — embedding model

---
