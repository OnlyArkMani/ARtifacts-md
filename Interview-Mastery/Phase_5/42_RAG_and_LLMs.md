# 42_RAG_AND_LLM_INTEGRATION

1. Introduction

What this concept is

Retrieval-Augmented Generation (RAG) combines retrieval of context documents with a generative model to ground responses and reduce hallucinations.

Why it exists

To provide accurate, up-to-date, and evidence-backed responses from LLMs by injecting relevant documents at inference time.


2. Components

- Retriever (BM25, dense vector search)
- Reader/generator (LLM conditioned on retrieved context)
- Indexing pipeline and vector stores (FAISS, Milvus)


3. 5-min revision

RAG pipelines: index docs, embed queries, retrieve k candidates, condition LLM with retrieved context and prompt templates. Evaluate for latency and retrieval quality.