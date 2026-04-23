# System Architecture

## Overview
Lens has a modular pipeline structured around the Retrieval-Augmented Generation (RAG) framework, supplemented by offline LoRA fine-tuning for domain specialization.

## Modules

### Data Ingestion pipeline (`src/ingestion`)
- **arXiv Fetcher**: Accesses the arXiv API for papers within AI/CS subject areas.
- **PDF Parser**: Uses PyMuPDF to extract and clean textual data from papers.
- **Chunker**: Splits large documents structurally or purely by characters using LangChain's `RecursiveCharacterTextSplitter`.

### Indexing (`src/indexing`)
- **Embedder**: Converts text chunks into dense semantic vectors using Sentence-Transformers (e.g., `all-MiniLM-L6-v2`).
- **FAISS Store**: Local vector database mapping embeddings to chunk IDs for efficient $k$-NN search.

### Retrieval Module (`src/retrieval`)
- Implements cosine similarity vector search over the FAISS store, pulling the top-k chunks in response to a user query.

### Generation Module (`src/generation`)
- Runs a local instance of an LLM through Ollama (e.g. `phi3:mini` or `llama3`). Prompt templates are populated with the top-k retrieved chunks as operational context to ground generations and minimize hallucinations.

### Fine-tuning Module (`src/finetuning`)
- Collects and formats abstract-QA-answer pairs for LoRA adaptation.
- Automates PEFT training configurations using `Hugging Face transformers` and `peft`.

### Evaluation (`src/evaluation`)
- Measures ROUGE-L scores against GPT-4o-mini baseline text.
- Human Evaluation suite to score citations and synthesis quality.
