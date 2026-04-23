# AutoResearch: Lens

**Lens** is a locally hosted, LoRA Fine-Tuned Retrieval-Augmented Generation (RAG) System designed for Computer Science and Artificial Intelligence (CS/AI) Research Paper Discovery and Synthesis. 

This repository contains the full source code for the project, allowing users to:
1. Ingest papers from arXiv.
2. Build local FAISS indices using chunked, semantic embeddings.
3. Query the collection using local LLMs configured via Ollama.
4. Perform local parameter-efficient fine-tuning (LoRA) of local models on a curated dataset.

## Installation and Setup
Refer to the `docs/setup_guide.md` for full installation instructions.

## Quick Start
* Coming soon...

## Project Structure
- `data/` : Raw and processed PDFs, and the FAISS index.
- `dataset/` : Dataset configured for LoRA training.
- `docs/` : Project proposal and internal architecture details.
- `src/` : Core RAG, ingestion, and fine-tuning architecture pipelines.
- `notebooks/` : Notebooks containing data exploration.

## License
MIT
