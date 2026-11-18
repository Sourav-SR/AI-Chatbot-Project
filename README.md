# DocuMind: Context-Aware Document Intelligence Assistant

DocuMind is an end-to-end Conversational Retrieval-Augmented Generation (RAG) platform that enables intelligent question-answering over personal and enterprise documents. Built with a decoupled microservice architecture (FastAPI backend + Streamlit frontend), it ingests unstructured documents, constructs semantic vector indexes, and delivers grounded, hallucination-resistant answers with multi-turn conversational context.

---

## Architecture Overview

```
User Document (.pdf, .docx, .txt)
        │
        ▼
[Document Loader & Text Extractor]
        │
        ▼
[Recursive Text Splitter (1000 chars / 200 overlap)]
        │
        ▼
[SentenceTransformer Embeddings (all-MiniLM-L6-v2)]
        │
        ▼
[ChromaDB Local Vector Storage]
        ▲
        │ Similarity Search (Top-K)
        │
[Contextual Query Reformulation] ◄── User Follow-up Question + Session Memory
        │
        ▼
[LLM Synthesis Engine (Groq / Qwen / LLaMA)]
        │
        ▼
Streamlit Interactive UI (Answer + Grounded Source Metadata)
```

---

## Core Capabilities

- **Multi-Format Ingestion**: Native extraction for Adobe PDF (`.pdf`), Microsoft Word (`.docx`), and standard plain text (`.txt`).
- **Semantic Chunking & Embedding**: Utilizes HuggingFace `sentence-transformers/all-MiniLM-L6-v2` for dense vector representations, running entirely offline.
- **Persistent Knowledge Base**: Local vector persistence powered by **ChromaDB**, ensuring data privacy and zero cloud storage dependency.
- **Conversational Memory**: Stateful session management tracking dialogue history to accurately resolve conversational context across follow-up queries.
- **Context-Aware Query Rewriting**: Automatically reformulates ambiguous follow-up questions into standalone retrieval queries prior to vector lookup.
- **Source Attribution**: Answers cite source documents and chunk indexes for complete traceability and auditability.

---

## Project Structure

```
├── backend/
│   ├── app/
│   │   ├── core/           # Environment and path configurations
│   │   ├── models/         # Pydantic schemas and request/response models
│   │   ├── routes/         # REST API route handlers (chat, upload, health)
│   │   ├── services/       # RAG pipeline, LLM orchestration, document parsing
│   │   ├── utils/          # File handling utilities
│   │   └── main.py         # FastAPI application entry point
│   ├── requirements.txt    # Backend service dependencies
│   └── .env                # Runtime secrets and environment variables
│
├── frontend/
│   ├── app.py              # Streamlit interactive web interface
│   └── requirements.txt    # Frontend client dependencies
│
├── chroma_db/              # Persistent local vector database storage
└── README.md
```

---

## Installation & Setup

### Prerequisites
- Python 3.11+
- Groq API credentials

### 1. Backend Service Configuration

Navigate to the `backend` folder and initialize the environment:

```bash
cd backend
python -m venv .venv

# Activate environment
# On Windows:
.\.venv\Scripts\activate
# On Linux/macOS:
# source .venv/bin/activate

pip install -r requirements.txt
```

Create `.env` in the `backend/` directory:

```env
GROQ_API_KEY=your_groq_api_key_here
GROQ_MODEL=qwen/qwen3.8-27b
APP_ENV=dev
```

Start the FastAPI backend server:

```bash
# From workspace root
uvicorn backend.app.main:app --reload --port 8000
```

The backend API service will be accessible at `http://localhost:8000`.

---

### 2. Frontend Web Interface

In a separate terminal, set up and launch the frontend client:

```bash
cd frontend
python -m venv .venv

# Activate environment
# On Windows:
.\.venv\Scripts\activate
# On Linux/macOS:
# source .venv/bin/activate

pip install -r requirements.txt
streamlit run app.py --server.port 8501
```

Access the application dashboard at `http://localhost:8501`.

---

## REST API Specification

### Health Check
- **`GET /health`**
  - Verifies server operational status.
  - Response: `{"status": "ok"}`

### Document Ingestion
- **`POST /upload`**
  - Content-Type: `multipart/form-data`
  - Parameters: `file` (Binary payload: `.pdf`, `.docx`, `.txt`)
  - Response:
    ```json
    {
      "filename": "sample.pdf",
      "chunks_added": 14,
      "message": "File ingested successfully."
    }
    ```

### Contextual Chat
- **`POST /chat`**
  - Content-Type: `application/json`
  - Payload:
    ```json
    {
      "session_id": "session-unique-identifier",
      "message": "Summarize the primary conclusions of the document."
    }
    ```
  - Response:
    ```json
    {
      "answer": "...",
      "metadata": {
        "retrieved": true,
        "sources": [
          {"source": "sample.pdf", "chunk_index": 2}
        ],
        "standalone_query": "..."
      }
    }
    ```

---

## Security & Privacy Notice
- Embedding generation and vector operations execute entirely on local hardware.
- Only the retrieved text context necessary to answer the active prompt is transmitted to the inference engine.
- Secrets and vector data directories are excluded from source control via `.gitignore`.
