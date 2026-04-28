# 📊 Financial Earnings Call Assistant

A production-ready **Retrieval-Augmented Generation (RAG)** system for analyzing SEC filings (10-K, 10-Q) using hybrid retrieval, cross-encoder re-ranking, and LLM-powered responses.

---

## 🏗️ Architecture Overview

```
User Query
    │
    ▼
FastAPI Layer (/query, /search, /ingest)
    │
    ▼
Hybrid Retrieval (FAISS Dense + BM25 Sparse)
    │
    ▼
Cross-Encoder Re-ranking
    │
    ▼
Context Injection → LLM (OpenAI / Mistral)
    │
    ▼
Grounded Response with Citations
```

---

## 📁 Project Structure

```
financial-earnings-assistant/
├── app/
│   ├── api/
│   │   ├── routes/
│   │   │   ├── query.py          # /query endpoint
│   │   │   ├── search.py         # /search endpoint
│   │   │   └── ingest.py         # /ingest endpoint
│   │   └── middleware.py         # CORS, logging, caching
│   ├── core/
│   │   ├── config.py             # Settings and env vars
│   │   └── logging.py            # Structured logging
│   ├── ingestion/
│   │   ├── sec_fetcher.py        # SEC EDGAR API client
│   │   ├── html_parser.py        # HTML → clean text
│   │   └── section_extractor.py  # Extract MD&A, Risk, etc.
│   ├── preprocessing/
│   │   ├── cleaner.py            # Text normalization
│   │   └── chunker.py            # Semantic chunking
│   ├── embeddings/
│   │   ├── encoder.py            # Sentence-transformer embeddings
│   │   └── faiss_store.py        # FAISS index management
│   ├── retrieval/
│   │   ├── dense.py              # FAISS dense retrieval
│   │   ├── sparse.py             # BM25 sparse retrieval
│   │   ├── hybrid.py             # Weighted hybrid fusion
│   │   └── reranker.py           # Cross-encoder re-ranking
│   ├── generation/
│   │   ├── llm_client.py         # OpenAI / Mistral client
│   │   └── prompt_builder.py     # Context injection & prompts
│   ├── evaluation/
│   │   ├── ragas_eval.py         # RAGAS evaluation pipeline
│   │   └── query_dataset.py      # 200+ financial queries
│   ├── models/
│   │   ├── request.py            # Pydantic request models
│   │   └── response.py           # Pydantic response models
│   └── main.py                   # FastAPI app entrypoint
├── data/
│   ├── raw/                      # Downloaded SEC filings
│   ├── processed/                # Cleaned text chunks
│   └── sample/                   # Sample dataset
├── storage/
│   ├── faiss_index/              # FAISS vector index
│   └── bm25_index/               # BM25 index pickle
├── tests/
│   ├── test_ingestion.py
│   ├── test_retrieval.py
│   └── test_api.py
├── postman/
│   └── Financial_Earnings_Assistant.postman_collection.json
├── docker/
│   ├── Dockerfile
│   └── docker-compose.yml
├── scripts/
│   ├── ingest_all.py             # Bulk ingest 20+ companies
│   └── run_evaluation.py         # Run RAGAS evaluation
├── requirements.txt
├── .env.example
└── README.md
```

---

## 🚀 Quick Start

### 1. Clone & Install

```bash
git clone <repo>
cd financial-earnings-assistant
pip install -r requirements.txt
```

### 2. Configure Environment

```bash
cp .env.example .env
# Edit .env with your API keys
```

### 3. Ingest SEC Data (5 companies)

```bash
python scripts/ingest_all.py
```

### 4. Run the API Server

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### 5. Access API Docs

Open: http://localhost:8000/docs

---

## 🐳 Docker Setup

```bash
docker-compose up --build
```

---

## 📡 API Endpoints

### POST `/query`
Returns an LLM-generated answer with citations.

```json
{
  "question": "What were Apple's revenue growth drivers in 2023?",
  "company_filter": "AAPL",
  "year_filter": 2023,
  "section_filter": "MD&A",
  "filing_type": "10-K",
  "top_k": 5
}
```

**Response:**
```json
{
  "answer": "Apple's revenue growth in 2023 was driven by...",
  "citations": [
    {
      "company": "AAPL",
      "year": 2023,
      "quarter": null,
      "filing_type": "10-K",
      "section": "MD&A",
      "chunk_id": "aapl_2023_10k_mda_001",
      "relevance_score": 0.94
    }
  ],
  "confidence": 0.91,
  "cached": false
}
```

### POST `/search`
Returns raw retrieved document chunks.

```json
{
  "query": "Tesla risk factors 2023",
  "company_filter": "TSLA",
  "top_k": 10,
  "retrieval_mode": "hybrid"
}
```

### POST `/ingest`
Triggers ingestion for a specific company.

```json
{
  "ticker": "MSFT",
  "filing_types": ["10-K", "10-Q"],
  "years": [2022, 2023, 2024]
}
```

---

## 🏢 Supported Companies (20+)

| Ticker | Company | Ticker | Company |
|--------|---------|--------|---------|
| AAPL | Apple | MSFT | Microsoft |
| GOOGL | Alphabet | AMZN | Amazon |
| META | Meta | TSLA | Tesla |
| NVDA | NVIDIA | JPM | JPMorgan Chase |
| BAC | Bank of America | WMT | Walmart |
| JNJ | Johnson & Johnson | PG | Procter & Gamble |
| XOM | ExxonMobil | CVX | Chevron |
| HD | Home Depot | DIS | Disney |
| NFLX | Netflix | CRM | Salesforce |
| AMD | AMD | INTC | Intel |
| V | Visa | MA | Mastercard |
| UNH | UnitedHealth | PFE | Pfizer |

---

## 📊 Evaluation (RAGAS)

```bash
python scripts/run_evaluation.py
```

Evaluates on 200+ financial queries across:
- **Answer Correctness** — factual accuracy
- **Faithfulness** — grounded in retrieved context
- **Context Precision** — retrieval quality

---

## ⚙️ Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENAI_API_KEY` | OpenAI API key | Required |
| `EMBEDDING_MODEL` | Sentence transformer model | `all-MiniLM-L6-v2` |
| `RERANKER_MODEL` | Cross-encoder model | `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| `FAISS_INDEX_PATH` | Path to FAISS index | `storage/faiss_index` |
| `LLM_PROVIDER` | `openai` or `mistral` | `openai` |
| `LLM_MODEL` | Model name | `gpt-4o-mini` |
| `CHUNK_SIZE` | Token chunk size | `750` |
| `CHUNK_OVERLAP` | Token overlap | `100` |
| `TOP_K_RETRIEVAL` | Chunks to retrieve | `10` |
| `TOP_K_RERANK` | Chunks after reranking | `5` |
| `REDIS_URL` | Redis for query caching | `redis://localhost:6379` |
| `SEC_USER_AGENT` | SEC EDGAR user agent | Required |
