# 🔭 Local Physics RAG Chatbot

An offline, local Retrieval-Augmented Generation (RAG) system acting as an undergraduate-level physics tutor. Powered by **Ollama (`qwen2.5:7b` + `nomic-embed-text`)**, **LangChain**, **ChromaDB**, and **Streamlit**.

The corpus consists of **OpenStax University Physics Volumes 1–3** (PDFs) and **The Feynman Lectures on Physics Volumes I–III** (HTML scraped with preserved LaTeX math).

---

## 🏗️ Architecture

```mermaid
flowchart TD
    A[User Query] --> B[Input Sanitiser\ntruncate 500 chars]
    B --> C{Domain Guard\nscore < 0.35?}
    C -- OOS --> D[Polite Refusal\nno LLM called]
    C -- Physics --> E[MMR Ensemble Retriever\nBM25 0.3 + Semantic 0.7]
    E --> F[CrossEncoder Reranker\n7 → top 5 chunks]
    F --> G{Empty Guard\nchunks == 0?}
    G -- Empty --> H[Corpus Error Message]
    G -- OK --> I[Retrieval Strength Score\nmean cosine similarity]
    I --> J[RAG Prompt Builder\nchunks + query]
    J --> K[qwen2.5:7b\ntemp=0 · max 512 tokens]
    K --> L[Citation Parser]
    L --> M[Streamlit UI\nAnswer + Sources + Strength Badge]

    subgraph Ingestion [Offline Ingestion Pipeline]
        N[OpenStax Vol 1-3 PDFs] --> P[PyMuPDF + pdfplumber\nMangle Detector]
        O[Feynman Vol I-III HTML] --> Q[BeautifulSoup\nLaTeX Preserved]
        P --> R[Sidebar Noise Filter\nRemove Check Your Understanding]
        R --> S[RecursiveCharTextSplitter\n1400/300 · 1200/250]
        Q --> S
        S --> T[nomic-embed-text\nOllama Embeddings]
        T --> U[(ChromaDB\nPersisted · Backed up)]
        U --> E
    end
````

---

## 🛠️ Installation & Setup

### 1. Prerequisites

* **Python 3.10 or 3.11**
* **Ollama** installed on your machine.

Pull the required local models:

```bash
ollama pull qwen2.5:7b
ollama pull nomic-embed-text
```

### 2. Setup Virtual Environment

Clone this repository and create a virtual environment:

```bash
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 3. Set up Environment Variables

Copy the template `.env.example` to `.env`:

```bash
cp .env.example .env
```

---

## 📥 Ingestion & Scraping

### 1. Download OpenStax PDFs

```bash
python scripts/download_corpus.py
```

### 2. Scrape Feynman Lectures

```bash
python scripts/feynman_scraper.py
```

### 3. Ingest Into ChromaDB

```bash
python src/ingest.py
```

---

## 🚀 Running the Web Application

Launch the Streamlit UI:

```bash
streamlit run app.py
```

Open:

```text
http://localhost:8501
```

---

## 🧪 Evaluation & Hallucination Test Suite

### 1. Run Baseline (Bare LLM)

```bash
python tests/baseline_runner.py
```

### 2. Run Hallucination & Retrieval Suite

```bash
python tests/hallucination_suite.py --report --baseline
```

---

## 🐳 Containerised Deployment

```bash
docker-compose up -d
```

Download models inside container:

```bash
docker exec -it physics-ollama ollama pull qwen2.5:7b
docker exec -it physics-ollama ollama pull nomic-embed-text
```

---

## ⚠️ Important Considerations & Design Decisions

### Retrieval Strength vs. Confidence

The Retrieval Strength badge shown in the UI is a proxy metric of retrieval relevance and NOT a calibrated probability of correctness.

### Concurrency & Performance

* Streamlit is single-threaded.
* Local reranking and embeddings may exhaust VRAM under concurrent requests.

### LaTeX Equation Degradation

The ingestion pipeline uses:

* PyMuPDF
* mangle detection
* pdfplumber fallback

to minimise equation corruption during PDF extraction.

```
```
