# COMP 3610: Big Data Analytics — Assignment 3
## LLM-Powered Applications & Distributed Computing

This project combines Apache Spark for large-scale data processing with a Retrieval-Augmented Generation (RAG) pipeline over NYC transportation policy documents, unified through an LLM-powered query router.

---

## Project Structure
```
.
├── notebook.ipynb          # Main Jupyter notebook (all code, outputs, analysis)
├── docs/                   # Curated PDF document corpus
│   ├── tlc_2025_annual_report.pdf
│   ├── feb_fhv_license_review_2025.pdf
│   ├── factbook_2020.pdf
│   ├── 2024_OFS_Annual_Report.pdf
│   ├── tlc_fare_rule_package_2024.pdf
│   ├── taxi_improvement_fund_2024.pdf
│   ├── congestion_pricing.pdf
│   ├── first_q_2019_report.pdf
│   ├── 2021_office_inclusion_annual_report.pdf
│   └── impact_of_congestion.pdf
├── requirements.txt        # Python dependencies
├── README.md               # This file
└── .gitignore              # Excludes data files, chroma_db/, __pycache__/
```

> **Note:** The NYC Yellow Taxi Parquet file (`data/`) and ChromaDB collections (`chroma_db*/`) are excluded from version control. The notebook downloads the Parquet file automatically on first run.

---

## Dataset

- **Structured Data:** NYC Yellow Taxi Trip Records — January 2024 (Parquet)
  - Downloaded automatically from the TLC open data portal
  - ~2.96 million rows; columns include pickup/dropoff timestamps, locations, fares, tips, and distances

- **Document Corpus:** 10 publicly available PDFs related to NYC taxi and transportation policy (see `docs/`)

---

## Setup Instructions

### Prerequisites

- Python 3.10+
- Java 8 or 11 (required for PySpark)
- Google Colab (recommended) or a local Jupyter environment

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd <repo-folder>
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Run the notebook

Open `notebook.ipynb` in Jupyter or Google Colab and run all cells from top to bottom. The notebook will:

1. Download the NYC Yellow Taxi Parquet file to `data/raw/` (skipped if already present)
2. Download all 10 PDF documents to `docs/`
3. Run all Spark cleaning, feature engineering, and SQL analytics
4. Build and persist ChromaDB vector stores
5. Run the RAG pipeline and integrated query router

### 4. LLM Server

The notebook connects to a course-hosted LLM server (`llama3.3-70b-instruct`). If running outside the course environment, update the `LLM_BASE_URL` and `LLM_API_KEY` variables in the notebook to point to any OpenAI-compatible API endpoint (e.g., OpenAI, Groq, Ollama).

---

## Parts Overview

| Part | Description | Marks |
|------|-------------|-------|
| Part 1 | Distributed Data Processing with Spark | 30 |
| Part 2 | RAG Pipeline over Transportation Documents | 35 |
| Part 3 | Integrated Analytics Application | 25 |
| Part 4 | Documentation & Code Quality | 10 |

### Part 1 — Spark
- SparkSession configured with AQE, memory settings, and local mode
- Data cleaned (null removal, invalid trip filtering, ~95k rows removed)
- Derived columns: `trip_duration_minutes`, `trip_speed_mph`, `pickup_hour`, `pickup_day_of_week`, `tip_percentage`
- 5 Spark SQL queries using window functions, running totals, and CASE expressions
- Performance: caching ( approximately 3.9× speedup), partitioned Parquet write by `pickup_hour`, execution plan explained

### Part 2 — RAG Pipeline
- 10 PDF documents loaded via `PyPDFDirectoryLoader`; 114 valid pages after filtering
- Chunked with `RecursiveCharacterTextSplitter` (size=1000, overlap=200) → 334 chunks
- Embeddings: `all-MiniLM-L6-v2` via `sentence-transformers`; stored in ChromaDB (persisted to disk)
- Chunk size experiment: 500, 1000, and 2000 characters compared across 3 sample queries
- RAG pipeline: retrieve -> augment -> generate, with source citations
- Evaluation: 10 manually-verified QA pairs; 60% overall accuracy; retrieval and generation failures classified

### Part 3 — Integrated Application
- **Query Router:** LLM classifies queries as DATA / DOCUMENT / HYBRID (JSON output); 66.7% accuracy on 15-query test set
- **Data Query Handler:** Natural language-> Spark SQL-> natural language answer (LLM-to-SQL with retry-on-error)
- **End-to-End Demo:** 6 queries (2 per category) processed through the full pipeline; HYBRID queries combine both backends

---

## Key Dependencies
```
pyspark
pandas
langchain
langchain-community
langchain-text-splitters
pypdf
sentence-transformers
chromadb
openai
matplotlib
requests
```

See `requirements.txt` for pinned versions.

---

## AI Tool Disclosure

Claude (Anthropic) was used to assist with debugging, code explanation and documentation. 
NotebookLLM was used to understand the pdfs used for better creation of queries.
All submitted code was reviewed, tested, and understood
