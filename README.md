# 🔍 Hybrid CRAG RAG Pipeline

A research-grade RAG pipeline that implements and compares three retrieval strategies — Normal RAG, Hybrid RAG, and CRAG (Corrective RAG) — evaluated on RBI Annual Report 2024-25 with a custom LLM-based evaluator.

---

## 🧠 The Problem with Normal RAG

Standard RAG has two critical failure modes:
- **Low Recall** — dense retrieval alone misses relevant chunks that use different vocabulary
- **Illusion of Knowledge** — the LLM is forced to answer from whatever was retrieved, even if irrelevant

This project implements and benchmarks three increasingly sophisticated solutions.

---

## 🏗️ Pipeline Architecture

```
Query
  │
  ├──► Normal RAG
  │         └── ChromaDB (dense only) → LLM → Answer
  │
  ├──► Hybrid RAG
  │         └── FAISS (dense) + BM25 (sparse)
  │                   └── Manual combination + CrossEncoder Reranking → LLM → Answer
  │
  └──► CRAG (Corrective RAG)
            └── Hybrid Retrieval
                      └── Retrieval Evaluator (RELEVANT / IRRELEVANT / AMBIGUOUS)
                                └── RELEVANT   → use chunks directly
                                    IRRELEVANT → web search fallback (DuckDuckGo)
                                    AMBIGUOUS  → chunks + web search combined
                                          └── LLM → Answer
```

---

## ✨ Key Features

- **Normal RAG** — ChromaDB vector search baseline
- **Hybrid Retrieval** — FAISS dense + BM25 sparse combined manually, reranked with CrossEncoder
- **CRAG** — Retrieval evaluator that classifies each chunk as RELEVANT / IRRELEVANT / AMBIGUOUS and applies corrective web search
- **Custom Evaluator** — LLM-based faithfulness and relevancy scoring (no OpenAI dependency)
- **Comparison Table** — Side by side evaluation of all three pipelines across multiple queries

---

## 📊 Evaluation Results

Tested on RBI Annual Report 2024-25 with 3 queries of varying complexity:

| Query | Normal RAG F/R | Hybrid RAG F/R | CRAG F/R |
|-------|---------------|----------------|----------|
| Complex multi-section (GDP + Monetary Policy) | 1.00 / 0.00 | 0.90 / 0.90 | 1.00 / 1.00 |
| Single section (Food Inflation) | 1.00 / 1.00 | 1.00 / 1.00 | 1.00 / 1.00 |
| Single section (Electronic Imports) | 1.00 / 1.00 | 1.00 / 1.00 | 1.00 / 1.00 |

**F = Faithfulness, R = Relevancy (0 to 1)**

**Key finding:** Normal RAG completely fails on complex queries requiring information from multiple document sections (Relevancy = 0.00). Hybrid RAG and CRAG both handle these queries well, with CRAG achieving perfect scores.

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|------------|
| LLM | Groq (`llama-3.3-70b-versatile`) |
| Dense Retrieval | FAISS + HuggingFace Embeddings (`all-MiniLM-L6-v2`) |
| Sparse Retrieval | BM25 (`rank-bm25`) |
| Reranker | CrossEncoder (`ms-marco-MiniLM-L-6-v2`) |
| Vector Store | ChromaDB |
| Web Search Fallback | DuckDuckGo (`ddgs`) |
| PDF Parsing | pypdf + LangChain Text Splitter |
| Evaluation | Custom Groq-based LLM evaluator |

---

## 📁 Project Structure

```
hybrid-crag-rag-pipeline/
├── advanced_rag.ipynb     # Complete Colab notebook
├── data/
│   └── RBI_Annual_report.pdf   # Source document
├── .gitignore
└── requirements.txt
```

---

## 🚀 How to Run

### Option 1 — Google Colab (Recommended)
1. Open `advanced_rag.ipynb` in Google Colab
2. Upload `RBI_Annual_report.pdf` to `/content/`
3. Add your Groq API key using Colab Secrets (🔑 icon on left sidebar)
4. Run all cells sequentially

### Option 2 — Local
```bash
git clone https://github.com/YashAgarwalTheWiz/hybrid-crag-rag-pipeline.git
cd hybrid-crag-rag-pipeline
pip install -r requirements.txt
```

Set up `.env`:
```
GROQ_API_KEY=your_groq_api_key_here
```

## 🧠 How CRAG Works

```
1. Retrieve top-5 chunks using Hybrid RAG (FAISS + BM25 + Reranking)
2. For each chunk, send to LLM evaluator:
      → RELEVANT   : keep the chunk
      → IRRELEVANT : discard, trigger web search
      → AMBIGUOUS  : keep chunk AND trigger web search
3. If web search triggered → fetch top-3 DuckDuckGo results
4. Final context = relevant chunks + web results (if any)
5. Generate answer from final context
```

This corrective layer prevents the LLM from being forced to answer from irrelevant chunks — the core weakness of normal RAG.

---

## 📈 Key Findings

- Normal RAG fails on queries requiring information from **multiple document sections**
- Hybrid retrieval (FAISS + BM25 + reranking) significantly improves recall on complex queries
- CRAG adds a safety net — even when retrieval is imperfect, the evaluator catches irrelevant chunks and falls back to web search
- For simple single-section queries, all three methods perform equally — advanced retrieval only matters for complex queries

---

Download the RBI Annual Report 2024-25 from:
https://www.rbi.org.in/Scripts/AnnualReportPublications.aspx
Place it in the data/ folder as RBI_Annual_report.pdf


## 👤 Author

**Yash Agarwal**
Fresher @ TCS | ML/AI Engineer in progress
[GitHub](https://github.com/YashAgarwalTheWiz)
