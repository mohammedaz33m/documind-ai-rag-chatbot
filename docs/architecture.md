# System Architecture
DocuMind AI follows a Retrieval Augmente Generation (RAG) pipeline.

# Flow
┌─────────────────────────────────────────────────────────┐
│                    USER UPLOADS PDF                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              DOCUMENT PROCESSING                         │
│  1. Load PDF (PyPDFLoader)                              │
│  2. Split into chunks (RecursiveCharacterTextSplitter)  │
│  3. Convert to embeddings (HuggingFace)                 │
│  4. Store in vector DB (FAISS)                          │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  USER ASKS QUESTION                      │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              RETRIEVAL PIPELINE                          │
│  1. Convert question to embedding                        │
│  2. Search FAISS for similar chunks (MMR)               │
│  3. Rerank with CrossEncoder                            │
│  4. Return top K chunks                                  │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              GENERATION PIPELINE                         │
│  1. Build prompt with:                                   │
│     - Retrieved chunks (context)                         │
│     - Chat history                                       │
│     - User question                                      │
│  2. Send to Groq LLM                                    │
│  3. Get response                                         │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│              POST-PROCESSING                             │
│  1. Calculate confidence score                           │
│  2. Save to conversation memory                          │
│  3. Display with source citations                        │
└─────────────────────────────────────────────────────────┘

# Key Components

Retriever
Uses FAISS vector similarity search to retrieve relevant document chunks.

Reranker
Uses BAAI BGE CrossEncoder to improve relevance ordering.

Context Builder
Top ranked chunks are combined into a context window.

LLM
Groq-hosted LLaMA model generates the final answer.