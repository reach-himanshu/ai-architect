# Data Engineering for AI Module

You can't retrieve what you can't clean. This module covers the data pipeline patterns that feed embeddings, RAG, and fine-tuning.

## 🚀 Roadmap

### 1. Data Sourcing for AI
- Corpora and public datasets
- Licensing and usage rights
- Crawling and scraping considerations

### 2. Ingestion Pipelines (Batch & Streaming)
- Batch ingestion patterns
- Streaming ingestion patterns
- Prereq: `03-design-patterns/12_etl`, `03-design-patterns/16_streaming`

### 3. Parsing & Extraction
- PDF extraction (text, tables, layout)
- HTML cleaning
- OCR for scanned docs
- Code and structured data

### 4. Deduplication
- Exact dedup (hash-based)
- Near-dup (MinHash, SimHash, LSH)
- Prereq: `02-dsa/06_hash_tables`

### 5. PII Detection & Redaction
- Regex baselines
- NER-based (Presidio, spaCy)
- LLM-based redaction

### 6. Synthetic Data Generation with LLMs
- Instruction synthesis
- Preference pair generation
- Quality filtering

### 7. Human Labeling Workflows
- Label Studio, Argilla
- Inter-rater agreement basics
- Active learning loops

### 8. Data Versioning for AI Assets
- DVC, LakeFS
- Dataset lineage
- Reproducibility requirements

### 9. Quality Metrics & Data Cards
- Coverage, balance, leakage checks
- Data cards and documentation
- Red flags in AI training data

### 10. Feature Stores for AI
- Feast, Tecton overview
- When they help vs hurt for LLM apps
