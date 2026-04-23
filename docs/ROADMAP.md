# AutoResearch: Lens Project Roadmap

This document outlines the comprehensive roadmap for developing the Lens project. It breaks down the system into sequential phases corresponding to your core objectives, detailing **what needs to be done**, **how to implement it**, and **critical technical considerations** for each phase.

---

## Phase 1: Data Ingestion Pipeline
**Goal:** Fetch papers from arXiv, extract their text securely, and split them into manageable semantic chunks.

### What to do
1. Retrieve papers related to CS/AI domains using the arXiv API.
2. Extract clean text from the downloaded PDF papers.
3. Split the extracted text into smaller, meaningful chunks to prepare for vector embedding.

### How to implement
- **arXiv Fetching:** Use Python's `requests` combined with the `arxiv` package to search for categories like `cs.AI` or `cs.LG`. Target a batch of 200–500 papers. Save the PDFs locally to `data/raw/`.
- **Text Extraction:** Use `PyMuPDF` (imported as `fitz`) to read the PDFs computationally. Write a parser script in `src/ingestion/parser.py` that iterates over pages and strips out headers, footers, and unreadable characters. Save cleaned text to `data/processed/`.
- **Chunking:** Use LangChain's `RecursiveCharacterTextSplitter` in `src/ingestion/chunker.py`. Set a reasonable chunk size (e.g., 500-1000 tokens) with an overlap of 10-15% (e.g., 100-200 characters) to ensure no context is lost across chunk boundaries.

### What to consider
- **Rate Limiting:** Provide sleep delays between arXiv API requests to prevent being IP-banned.
- **Data Quality:** PDFs often contain mathematical formulas, tables, and multi-column layouts that break standard text extraction. PyMuPDF handles columns well, but keep an eye out for noisy text and clean it using Regex if necessary.

---

## Phase 2: Vector Embedding and FAISS Indexing
**Goal:** Convert parsed document chunks into dense vectors and store them in a searchable database.

### What to do
1. Generate mathematical embeddings (dense vectors) for every text chunk.
2. Store the embeddings in a FAISS index to allow for efficient $k$-Nearest Neighbour mathematical search.

### How to implement
- **Embedding Generation:** Use `sentence-transformers` via `src/indexing/embedder.py`. Load the model `all-MiniLM-L6-v2`. Create a script that converts all JSON chunk objects from Phase 1 into semantic embeddings.
- **FAISS Storage:** Use the `faiss-cpu` library. Inside `src/indexing/faiss_store.py`, initialize an `IndexFlatL2` (for cosine similarity/Euclidean distance search). Ensure you maintain a metadata mapping (a dictionary mapping the FAISS ID back to the chunk text and paper title). Save the index to `data/index/`.

### What to consider
- **Dimensionality:** `all-MiniLM-L6-v2` produces 384-dimensional vectors. Ensure your FAISS index is initialized to exact dimensions (`d=384`).
- **Memory Context:** Depending on your hardware, processing 200+ papers at once can cause Out-Of-Memory (OOM) errors. Batch the embedding generation sizes (e.g., process 32 chunks at a time).

---

## Phase 3: Semantic Search and Base Validation (Local RAG)
**Goal:** Query the database and pass the retrieved information into a base LLM hosted via Ollama.

### What to do
1. Accept a natural language user query.
2. Retrieve the top-$k$ (e.g., top 5) most relevant embeddings.
3. Pipe the chunks into a prompt and ask Ollama (`phi3:mini`) to answer based **only** on the context.

### How to implement
- **Retrieval:** In `src/retrieval/searcher.py`, implement an endpoint that takes a search query, embeds it using the same `all-MiniLM-L6-v2` model, and executes an `index.search()` in FAISS to get the closest IDs.
- **RAG Generation:** In `src/generation/qa_pipeline.py`, use the Ollama Python endpoints or LangChain’s Ollama integration. Construct a rigid prompt like:
  > *"You are an academic researcher. Answer the question based ONLY on the following context. If you don't know, say you don't know. Context: {retrieved_chunks}"*
- **Base Testing:** Prepare a manually curated set of 30 test questions to validate that Precision@5 targets are hit without any fine-tuning.

### What to consider
- **Hallucinations:** Look closely for instances where the base model ignores the given context and fabricates standard LLM knowledge. Document these—they serve as your benchmark to prove why your LoRA fine-tuning (Phase 4) is important.

---

## Phase 4: LoRA Fine-Tuning a Local LLM
**Goal:** Train an abstraction on top of a local model so it excels specifically at academic synthesis and citation generation.

### What to do
1. Prepare a QA dataset using the abstract and extracted chunks.
2. Fine-tune the base model (e.g., Llama 3 8B or Phi-3 mini) efficiently using LoRA in Google Colab.
3. Export the completed LoRA adapter.

### How to implement
- **Dataset Creation (`dataset/qa_pairs.json`):** Generate a dataset of minimum 500 triplets containing: System Prompt, Human Question (e.g. "What is method X in paper Y?"), and the Ideal Answer (derived from the abstract/chunks). Structure this in ChatML or the specific instruct template syntax.
- **Model Training:** Use Google Colab (T4 or A100 GPU). Inside `notebooks/`, write a script using the `peft`, `trl`, and `transformers` packages to load standard Llama 3 in 4-bit precision (`bitsandbytes`). Apply LoRA configurations (e.g. $r=8$, $\alpha=16$) to freeze the base model and only train adapters.
- **Training Objectives:** Use SFTTrainer (Supervised Fine-tuning Trainer). Save the checkpoint containing adapter weights.

### What to consider
- **Compute constraints:** Full parameter tuning takes heavy compute. LoRA guarantees GPU efficiency, but configuring 4-bit quantization (QLoRA) is highly recommended.
- **Overfitting:** Monitor your training loss curves. If loss falls to zero rapidly, the model is memorising the abstracts rather than learning to synthesise data.

---

## Phase 5: Pipeline Integration and Evaluation
**Goal:** Merge the fine-tuned model back into the RAG pipeline and empirically prove its success over the base model.

### What to do
1. Connect the fine-tuned LoRA adapters with your offline retrieval system.
2. Evaluate and compare generation quality.
3. Polish the CLI application.

### How to implement
- **Integration:** Pull the LoRA adapters locally out of Google Colab. Reconfigure `src/generation/qa_pipeline.py` to load the base model + LoRA adapter instead of raw Ollama. 
- **Automated Logging:** Write ROUGE-L scripts in `src/evaluation/evaluator.py`. Compare the responses of your system against OpenAI implementations (GPT-4o API baseline).
- **Human Rubric:** Apply identical questions to the Base Model, the LoRA Model, and the Cloud Baseline and have humans test readability, coherence, and accuracy. Track these in a spreadsheet.

### What to consider
- **Statistical Significance:** Ensure the ROUGE-L comparison dataset is well isolated. Avoid testing on papers the model was explicitly trained on to verify genuine semantic retrieval abstraction.
- **Report Writing:** The human metrics, cost justifications (Cloud API vs. Free Local), and evaluation rubrics will formulate the entirety of your eventual project report. Keep strict documentation of when your model fails vs. succeeds.
