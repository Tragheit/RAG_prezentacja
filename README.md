# RAG Prezentacja (RAG Presentation)

This project is a comprehensive demonstration and educational resource focused on **Retrieval-Augmented Generation (RAG)** and the power of **Vector Embeddings**. It explores how high-dimensional vector spaces can be used for both semantic search and logical analogies.

## Project Structure

- **`rag_prezentacja.ipynb`**: The main notebook demonstrating the RAG workflow, including document loading, chunking, embedding, and retrieval-based generation.
- **`country_capital_vector.ipynb`**: An exploration of vector arithmetic (e.g., *France - Paris + England = London*). This notebook demonstrates how to perform word analogies using OpenAI's `text-embedding-ada-002` model.
- **`documents/`**: A collection of technical guides in Markdown format:
  - `mierzenie_ir.md`: Measuring Information Retrieval performance.
  - `strategie_podzialu_dokumentow.md`: Strategies for document chunking.
  - `strategie_wektoryzacji_dokumentow.md`: Strategies for document vectorization.
  - `strategie_generowania_odpowiedzi.md`: Strategies for generating responses in RAG.
- **`data/`**: Contains pre-computed embeddings and subsets of word vectors for the analogy demonstrations.
- **`media/`**: Diagrams and visualizations of RAG architectures and document processing workflows.

## Features

- **Vector Analogies**: Improved prediction of country capitals using averaged relationship vectors from multiple reference pairs.
- **Document Processing**: Examples of different chunking and vectorization strategies.
- **Interactive Visualization**: 3D plots of word embeddings to visualize clusters and relationships.

## Setup

1. **Environment**: It is recommended to use a virtual environment.
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

2. **Dependencies**: Install the required packages.
   ```bash
   pip install -r requirements.txt
   ```

3. **API Keys**: Copy `.env_template` to `.env` and provide your `OPENAI_API_KEY`.
   ```bash
   cp .env_template .env
   ```

## Note on Development
This project was refined and optimized with the assistance of **Gemini CLI**, an interactive AI agent for software engineering.
