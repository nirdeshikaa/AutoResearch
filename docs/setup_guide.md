# Setup Guide

## Prerequisites
- Python 3.10+
- Git
- Ollama (for running local models)

## Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/nirdeshikaa/AutoResearch.git
   cd AutoResearch
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows, use .venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Install Ollama & Download Models:**
   - Download Ollama from: https://ollama.com/
   - Start the Ollama server.
   - Pull the base model, for example `phi3:mini` or `llama3`:
     ```bash
     ollama pull phi3:mini
     ```

## Usage
The command line application is under development.
