# Agentic Document Assistant

An advanced, agentic RAG (Retrieval-Augmented Generation) system for interacting with and analyzing documents, powered by LangGraph, Streamlit, and Qdrant. 

This system goes beyond basic keyword search. It leverages a dynamic agent loop to actively verify claims against real-time web data and handle multi-session contexts seamlessly.

---

## 🌟 Core Capabilities

*   **Intelligent Routing**: Automatically categorizes queries into Direct Answers, Document Retrieval, or Web Verification.
*   **Fact-Checking**: Validates claims in your documents by cross-referencing live web data (via Tavily).
*   **Self-Healing Retrieval**: Employs relevancy grading. If retrieved context is poor, the agent re-writes the query and tries again.
*   **Isolated Sessions**: Work on multiple documents in separate sessions without cross-contamination. Uses Qdrant for vector storage and SQLite for state persistence.
*   **Off-topic Queries**: Use the `/btw` command to ask quick questions without polluting your current document's context history.

## 🏗️ Architecture

The system is built on a cyclical state machine using LangGraph. User queries are routed through specialized nodes:
1.  **Direct Answer**: For general greetings or simple questions.
2.  **Retrieval**: Fetches relevant context from embedded documents.
3.  **Claim Verification**: Triggers web searches for fact-checking.

All embeddings are cached locally to reduce API costs, and conversational state is securely check-pointed in SQLite.

## 🚀 Getting Started

### Prerequisites
* Python 3.10+
* A Qdrant Cloud cluster
* API keys for Google AI Studio (Gemini) and Tavily

### Installation

1.  **Clone & Setup Environment**
    ```bash
    git clone https://github.com/yourusername/agentic-doc-assistant.git
    cd agentic-doc-assistant
    python -m venv venv
    
    # Windows
    .\venv\Scripts\activate
    # macOS/Linux
    source venv/bin/activate
    
    pip install -r requirements.txt
    ```

2.  **Configuration**
    Create a `.env` file in the project root:
    ```env
    GOOGLE_API_KEY="your_google_api_key"
    TAVILY_API_KEY="your_tavily_key"
    QDRANT_URL="your_qdrant_cluster_url"
    QDRANT_API_KEY="your_qdrant_api_key"
    APP_PASSWORD="your_ui_password"
    ```

3.  **Run the App**
    ```bash
    streamlit run gui.py
    ```
    Access the UI at `http://localhost:8501`.

## 🐳 Docker Deployment

To run the application via Docker:

```bash
# Setup persistent storage
mkdir -p ~/app-data/embedding_cache
touch ~/app-data/sessions.json
touch ~/app-data/checkpoints.db

# Build and Run
docker build -t agentic-doc-assistant .
docker run -d \
  -p 8501:8501 \
  --env-file .env \
  -v ~/app-data/embedding_cache:/app/embedding_cache \
  -v ~/app-data/sessions.json:/app/sessions.json \
  -v ~/app-data/checkpoints.db:/app/checkpoints.db \
  --restart unless-stopped \
  --name doc-assistant-app \
  agentic-doc-assistant:latest
```

## 🧪 Evaluation

Run the automated evaluation suite (powered by DeepEval) to test the RAG pipeline's Faithfulness, Relevancy, and Precision:

```bash
python run_evals.py
```

## 📝 Notes
* Keep an eye on API quotas if processing very large documents.
* Vector collections are isolated per session for privacy and accuracy.
