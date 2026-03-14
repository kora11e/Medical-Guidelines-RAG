## RAG Project 

A Retrieval-Augmented Generation (RAG) system for answering medical questions using PDF medical guidelines. The system retrieves relevant document fragments and uses the Bielik-11B-v3.0-Instruct LLM to generate answers grounded in those [documents](https://ptkardio.pl/wytyczne). 

Overview

This project implements a local RAG pipeline consisting of:

- Document ingestion (PDF medical guidelines)

- Text chunking

- Vector embeddings

- FAISS vector database

- Context retrieval

- LLM answer generation

The system ensures answers are grounded in the provided documents, reducing hallucinations.

## Models Used
### Embedding Model

intfloat/multilingual-e5-base

Multilingual sentence embedding model

Used to convert document chunks and queries into vectors

### Language Model

speakleash/Bielik-11B-v3.0-Instruct

Polish instruction-tuned LLM loaded in 4-bit quantization using BitsAndBytes. Generates final answers using retrieved context.

## Project Pipeline
1. PDF Loading

The system reads all PDF files from a dataset directory and extracts text.

Function:

load_pdfs(folder_path)

Output:

[
  {
    "source": "file.pdf",
    "text": "full document text"
  }
]
2. Text Chunking

Documents are split into overlapping chunks to improve retrieval quality.

Parameters:

CHUNK_SIZE = 800
CHUNK_OVERLAP = 150

Function:

chunk_text()

Example chunk structure:

[
  "chunk1 text...",
  "chunk2 text..."
]
3. Chunk Preparation

Chunks are paired with metadata indicating their source document.

Function:

prepare_chunks(pdf_docs)

Output:

chunks: list[str]
metadata: [{"source": "document.pdf"}]
4. Embedding & Vector Index

Each chunk is converted into embeddings and stored in a FAISS vector index.

Function:

build_vector_index(chunks)

Steps:

Load embedding model

Generate embeddings for all chunks

Store vectors in FAISS IndexFlatL2

5. Query Retrieval

When a user asks a question:

Query is embedded

Similar vectors are retrieved from FAISS

Function:

retrieve(query, index, embedding_model, chunks, metadata, k)

Returns the top K relevant chunks.

TOP_K = 3
6. Prompt Construction

Retrieved chunks are combined into a prompt that instructs the model to answer only using provided context.

## Example prompt:

You are a medical assistant.
Answer only based on the provided context.
If the information is missing, say it was not found.

Context:
<retrieved text>

Question:
<user query>

Answer:

Function:

build_prompt()
7. Answer Generation

The final answer is generated using the Bielik LLM.

Function:

generate_answer()

Generation parameters:

max_new_tokens = 400
temperature = 0.3
top_p = 0.9
repetition_penalty = 1.1
