<div align="center">

# 🧠 Agentic Document Assistant

**An intelligent, self-correcting RAG system that doesn't just retrieve — it reasons, verifies, and adapts.**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![LangGraph](https://img.shields.io/badge/LangGraph-Orchestration-FF6F00?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain-ai.github.io/langgraph/)
[![Streamlit](https://img.shields.io/badge/Streamlit-UI-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)](https://streamlit.io)
[![Qdrant](https://img.shields.io/badge/Qdrant-Vector%20DB-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://qdrant.tech)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)


</div>

---

## 💡 The Problem

Most RAG chatbots can tell you **what a document says** — but they can't tell you **whether it's still true**. In fast-moving fields like AI and Machine Learning, a claim from six months ago may already be outdated.

**Agentic Document Assistant** solves this by treating _"Is this claim still valid?"_ as a first-class operation. Instead of simple single-hop retrieval, it uses an autonomous **LangGraph agent loop** that dynamically routes queries, challenges assertions against live web data, detects poor retrieval quality, and self-corrects on the fly.

---

## ✨ Features

| Feature | Description |
| :--- | :--- |
| 📄 **Multi-Source Ingestion** | Upload PDFs, Markdown, TXT files, scrape web URLs, or pull papers directly from **arXiv** by ID or title. |
| 🔀 **Autonomous Query Routing** | An agentic router classifies each query into **Direct Answer**, **Document Retrieval**, or **Claim Verification** — no manual mode-switching needed. |
| ✅ **Live Claim Verification** | Cross-references factual claims against recent publications via Tavily web search, highlighting outdated findings with citations. |
| 🔄 **Self-Correcting Retrieval** | A built-in relevancy grading node detects low-quality chunks and triggers automated **query rewrites** and retries to minimize hallucinations. |
| 🗂️ **Isolated Multi-Session Memory** | Each chat session maintains its own Qdrant vector collection and SQLite state checkpoint — zero context bleed across documents. |
| 💬 **`/btw` Side Channel** | Ask quick, off-topic questions without corrupting your active research session's conversational memory. |
| ⚡ **Disk-Cached Embeddings** | Content chunks are SHA-256 hashed and cached locally to avoid redundant, costly embedding API calls. |
| 🏷️ **Auto-Named Sessions** | An LLM generates concise, descriptive session titles from your initial query — no more "Untitled Chat". |

---

## 🏛️ System Architecture

```text
                                ┌───────────────────────────┐
                                │   User / Streamlit UI     │
                                └─────────────┬─────────────┘
                                              │
                                              ▼
                                ┌───────────────────────────┐
                                │   LangGraph Agent Router  │
                                └──────┬──────┬──────┬──────┘
                                       │      │      │
             ┌─────────────────────────┘      │      └────────────────────────┐
             ▼                                ▼                               ▼
   ┌────────────────────┐          ┌────────────────────┐          ┌────────────────────┐
   │ Direct Answer Node │          │  Retrieval Agent   │          │ Claim Verification │
   └────────────────────┘          └──────────┬─────────┘          └──────────┬─────────┘
                                              │                               │
                                    ┌─────────┴─────────┐                     ▼
                                    ▼                   ▼           ┌───────────────────┐
                          ┌──────────────────┐ ┌──────────────────┐ │ Tavily Web Search │
                          │  Qdrant Vector   │ │ Tavily Search    │ └───────────────────┘
                          │   Collection     │ │ (Web Context)    │
                          └─────────┬────────┘ └──────────────────┘
                                    │
                                    ▼
                         [ Relevancy Evaluation ]
                                    │
                       ┌────────────┴────────────┐
                       │ (Pass)                  │ (Fail)
                       ▼                         ▼
            ┌─────────────────────┐   ┌───────────────────────┐
            │   Generate Final    │   │ Query Rewrite & Retry │
            │ Grounded Answer     │   └──────────┬────────────┘
            └─────────────────────┘              │
                       ▲                         │
                       └─────────────────────────┘
```

> **State Persistence:** Conversational history, agent state checkpoints, and session metadata are persisted through an embedded SQLite checkpoint database alongside isolated Qdrant vector spaces.

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
| :--- | :--- | :--- |
| **Agent Framework** | LangGraph | Explicit cyclical graph state machine with dynamic routing, retries, and fallback handling. |
| **LLM & Embeddings** | Google Gemini | High-throughput `gemini-flash` for reasoning paired with native text embeddings. |
| **Vector Database** | Qdrant Cloud | Low-latency vector search with per-session collection isolation and payload filtering. |
| **Live Web Retrieval** | Tavily Search API | Optimized for agent workflows — discovers recent counter-evidence and fresh research. |
| **Evaluation** | DeepEval | LLM-assisted evaluation for Faithfulness, Answer Relevancy, and Contextual Precision. |
| **Frontend** | Streamlit | Fast, responsive chat UI with word-by-word streaming and instant source access. |
| **Deployment** | Docker + AWS EC2 | Fully containerized environment for cloud deployment. |

---

## 📂 Project Structure

```
.
├── gui.py                    # Streamlit application entry point & UI
├── core/
│   ├── agent_logic.py        # LangGraph state machine, agent nodes & dynamic router
│   ├── database.py           # Qdrant client, isolated collections & disk-caching
│   ├── ingestion.py          # Document loaders (PDF, Web scraping, arXiv API)
│   ├── offtopic.py           # /btw side-channel conversational memory logic
│   └── data_types.py         # Pydantic data schemas & state models
├── documents/                # Sample research papers
├── run_evals.py              # DeepEval testing harness
├── Dockerfile                # Production container image
├── DEPLOY_GUIDE.md           # Docker & cloud deployment guide
├── requirements.txt          # Python dependencies
└── README.md
```

---

## ⚡ Quick Start

### Prerequisites

- Python `3.10+`
- A [Qdrant Cloud](https://cloud.qdrant.io/) cluster
- API keys for [Google AI Studio](https://aistudio.google.com/) and [Tavily](https://tavily.com/)

### 1️⃣ Clone & Install

```bash
git clone https://github.com/shobhitranjan2005/Agentic-RAG.git
cd Agentic-RAG

python -m venv venv

# Windows
.\venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

pip install -r requirements.txt
```

### 2️⃣ Configure Environment

Create a `.env` file in the project root:

```env
GOOGLE_API_KEY="your_google_ai_studio_key"
TAVILY_API_KEY="your_tavily_search_key"
QDRANT_URL="https://your-cluster.cloud.qdrant.io"
QDRANT_API_KEY="your_qdrant_api_key"
APP_PASSWORD="your_secure_password"
```

### 3️⃣ Launch

```bash
streamlit run gui.py
```

Open your browser at **`http://localhost:8501`** and start asking questions! 🚀

---

## 🐳 Docker Deployment

```bash
# Setup persistent storage
mkdir -p ~/app-data/embedding_cache
touch ~/app-data/sessions.json
touch ~/app-data/checkpoints.db

# Build & Run
docker build -t agentic-doc-assistant .
docker run -d \
  -p 8501:8501 \
  --env-file .env \
  -v ~/app-data/embedding_cache:/app/embedding_cache \
  -v ~/app-data/sessions.json:/app/sessions.json \
  -v ~/app-data/checkpoints.db:/app/checkpoints.db \
  --restart unless-stopped \
  --name doc-assistant \
  agentic-doc-assistant:latest
```

> 📖 For detailed AWS EC2 deployment instructions, see [`DEPLOY_GUIDE.md`](DEPLOY_GUIDE.md).

---

## 🧪 Evaluation & Benchmarking

Run the automated evaluation suite (powered by **DeepEval**) to grade the RAG pipeline using LLM-as-a-Judge:

```bash
python run_evals.py
```

| Metric | What It Measures |
| :--- | :--- |
| **Faithfulness** | Are answers strictly grounded in retrieved context? |
| **Answer Relevancy** | Do responses directly address the user's question? |
| **Contextual Precision** | Are retrieved chunks dense in signal and noise-free? |

---

## ⚠️ Known Considerations

- **API Rate Limits** — Free-tier API keys may hit quota limits during heavy concurrent processing.
- **Qdrant Hibernation** — Free-tier Qdrant clusters may sleep after extended inactivity.
- **Session Cleanup** — Deleting a session from the UI currently retains historical checkpoint entries in SQLite.

---

<div align="center">

**Built with ❤️ using LangGraph, Gemini, and Qdrant**

⭐ If you find this project useful, consider giving it a star!

</div>
