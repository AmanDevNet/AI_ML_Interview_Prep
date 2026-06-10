# 🛠️ Technical Interview Preparation Hub
### *Curated Revision Handbooks, System Designs, and Coding Tests for AI, ML, GenAI, and Backend Engineers*

This repository is a clean, production-focused study resource designed to help engineers prepare for senior technical loops, system design sessions, and coding screens at top-tier tech companies. 

Instead of textbook theories or library wrapper tutorials, this hub focuses on **core engineering principles, system tradeoffs, hardware constraints, and framework-independent implementations**.

---

## 🗺️ Repo Contents & Handbooks

The repository is organized into three distinct, self-contained handbooks. You can dive directly into each folder using the links below:

### 1. [🧠 LLM & RAG Interview Handbook](./LLM_RAG_Interview_Handbook/)
A comprehensive guide covering large language models and retrieval systems from basic parameters to production systems.
* **Phase 1: LLM Fundamentals:** Tokenization, vocabulary sizes, embeddings, transformer architecture, attention mechanisms, pretraining, fine-tuning (SFT, RLHF, DPO), temperature, top-p, TTFT/ITL, latency, and model evaluation.
* **Phase 2: RAG Fundamentals:** Custom chunking, vector embeddings, similarity search (Cosine vs. Dot Product), hybrid search (BM25 + Dense), reranking (Cross-Encoders), context synthesis, citations, and evaluation (RAGAS).
* **Phase 3: RAG System Design Blueprints:** 10 detailed production blueprints (e.g., enterprise wikis, legal research assistants, medical Q&A, e-commerce search, and multi-tenant SaaS platforms).
* **Phase 4: Deep RAG Engineering Q&A:** In-depth engineering solutions for parsing tables, handling dirty PDFs, updating/deleting document indices, and handling document access control (ACLs).
* **Phase 5: LLM/RAG Coding & Pseudocode:** Pure, framework-independent Python implementations of text splitters, vector similarity, Cross-Encoder APIs, and streaming response generators.
* **Phase 6: 100 Q&A Bank:** Curated technical interview questions divided by difficulty (Beginner, Intermediate, Advanced, and Debugging).
* **Phase 7: 90-Minute Mock Exam:** A full mock test containing 31 scenario-based questions with conversational answer scripts and follow-up loops.
* **Phase 8: Perspective & Voice Practice Prompts:** Blueprints for passing screening rounds and two custom system prompts to practice the interview using **ChatGPT Voice Mode** (one for the Interviewer and one for the Candidate).

### 2. [📊 SQL Revision & Practice Handbook](./SQL_Revision_Handbook/)
A focused reference guide designed for data pipelines, backend queries, and coding loops.
* **Phase 1: SQL Foundations:** Relational database concepts, basic querying, aggregations, deep joins, subqueries, CTEs, window functions, and database performance/indexes.
* **Phase 2: Query Patterns:** 20 common SQL interview query patterns with step-by-step query construction.
* **Phase 3: Theory Q&A Bank:** 50 structured questions on database internals, optimization, locking, isolation levels, and storage.
* **Phase 4: Mock Coding Test:** A progressive 30-question coding test spanning basic selects to complex multi-join window operations with inline schemas, solutions, and query explanations.

### 3. [🐍 Python Revision Handbook](./Python_Revision_Handbook/)
A clean refresher on core Python internals, memory, and concurrency.
* **Phase 1: Python Core:** Variables, basic data structures, scoping rules, modules, exception handling, and file processing.
* **Phase 2: Advanced OOP & Internals:** Inheritance, polymorphism, decorators, custom iterators, generators, closures, dataclasses, context managers, and time/space complexity analysis.
* **Phase 3: Systems & Production Python:** The Global Interpreter Lock (GIL), multithreading vs. multiprocessing, async/await, descriptors, typing protocols, Pydantic data validation, and NumPy/Pandas internals.

---

## 🎯 How to Use This Repository

Depending on the specific role you are interviewing for, you can follow these tailored study paths:

| Study Path | Recommended Chapters to Focus On |
| :--- | :--- |
| **GenAI / RAG Specialist** | LLM & RAG Handbook (Phases 1-8) → focus heavily on custom coding in Phase 5 and system designs in Phase 3. |
| **General AI / ML Engineer** | Python Handbook (Phases 2-3) → LLM & RAG Handbook (Phases 1-3 & 6) → SQL Handbook (Phases 2 & 4). |
| **Backend AI / Data Engineer** | Python Handbook (Phases 1-3) → SQL Handbook (Phases 1-4) → LLM & RAG System Design (Phase 3). |

---

## 🎤 Practice in Real-Time (ChatGPT Voice Mode)

To help bridge the gap between reading and speaking, we have included interactive prompt blueprints in [Phase 8 of the LLM & RAG Handbook](./LLM_RAG_Interview_Handbook/Phase8/):

1. **Interviewer Prompt:** Configures ChatGPT to act as a strict senior technical interviewer. It will ask questions, conduct deep-dives on your projects, and ask you to share your screen/IDE to verbally walk through algorithms.
2. **Candidate Prompt:** Configures ChatGPT to act as you (the candidate). Paste your resume at the bottom of the prompt to hear how a senior engineer would talk through your projects, explain tradeoffs, and pivot when they don't know the exact answer to a technical question.

---

## 🚀 Upcoming Roadmap (Coming Soon)

To keep this preparation hub aligned with modern production architectures, the following topics are actively being developed and will be added soon:

*   **🔌 Model Context Protocol (MCP) Integration:** Blueprints and custom Python server templates to securely connect LLMs with custom databases, file systems, and development tools.
*   **⚡ High-Performance FastAPI Deployments:** Async request optimization, streaming-response serialization, low-overhead Pydantic v2 validation pipelines, and background task execution.
*   **🤖 Multi-Agent Systems & Tool Calling:** Production design patterns for agentic workflows (router, orchestrator-workers, and reflection loops) along with structured JSON decoding validation.
*   **💾 Vector Database Scaling & Sharding:** Production deployments for million-scale vectors, performance analysis of index types (HNSW vs. IVF-PQ), and handling dynamic index updates without query downtime.

