# 🧠 LLM & RAG Technical Interview Handbook
### *The Ultimate Guide for Senior AI, ML, GenAI, and Backend AI Engineers*

This handbook is a production-focused reference guide designed to prepare senior engineers for technical screenings, system design rounds, and deep engineering loops in LLM and Retrieval-Augmented Generation (RAG) roles.

---

## 🗺️ Curriculum Roadmap

### [Phase 1: LLM Fundamentals](./Phase1/)
*Core model architecture, training loops, alignment methods, inference mechanics, and safety systems.*
* **Chapter 1: Core Architecture** ([01_Core_Architecture.md](./Phase1/01_Core_Architecture.md)) — *LLM definition, Tokenization, Vocab, Embeddings, Context Windows, Transformers, Attention, and Pretraining.*
* **Chapter 2: Alignment & Tuning** ([02_Alignment_Tuning.md](./Phase1/02_Alignment_Tuning.md)) — *Fine-Tuning (SFT), Instruction Tuning, and RLHF/DPO/PPO.*
* **Chapter 3: Inference & Prompting** ([03_Inference_Prompting.md](./Phase1/03_Inference_Prompting.md)) — *Prompting, System vs User prompts, Temperature, Top-p, and Max tokens.*
* **Chapter 4: Production Dynamics & Safety** ([04_Production_Evaluation.md](./Phase1/04_Production_Evaluation.md)) — *Hallucinations, Latency (TTFT/ITL), Cost tuning, Benchmarking/Evaluation, Structured outputs, Tool calling, and Guardrails.*

### [Phase 2: RAG Fundamentals](./Phase2/)
*Ingestion, Chunking strategies, Vector Embeddings, Search types, Reranking, and Evaluation loops.*
* **Chapter 1: RAG Core Foundations** ([01_RAG_Core_Foundations.md](./Phase2/01_RAG_Core_Foundations.md)) — *Core definitions, comparison to Fine-Tuning/Prompting, and pipeline overview.*
* **Chapter 2: Ingestion & Preprocessing** ([02_Ingestion_Preprocessing.md](./Phase2/02_Ingestion_Preprocessing.md)) — *Ingestion connectors, parsing, text cleaning, chunking strategies, and metadata.*
* **Chapter 3: Vectors & Embeddings** ([03_Vectors_Embeddings.md](./Phase2/03_Vectors_Embeddings.md)) — *Dense embeddings, vector databases, search similarity (Cosine vs. Dot Product), and Top-k.*
* **Chapter 4: Advanced Retrieval** ([04_Advanced_Retrieval.md](./Phase2/04_Advanced_Retrieval.md)) — *Hybrid search, BM25 keyword matching, reranking models, query rewriting, and multi-query.*
* **Chapter 5: Generation & Synthesis** ([05_Generation_Synthesis.md](./Phase2/05_Generation_Synthesis.md)) — *Context construction, prompt templates, citation frameworks, source attribution, and hallucination reduction.*
* **Chapter 6: Operations & Evaluation** ([06_Operations_Evaluation.md](./Phase2/06_Operations_Evaluation.md)) — *Document access control (ACLs), RAGAS metrics, offline Golden Datasets, and online feedback loops.*

### [Phase 3: RAG System Design Interviews](./Phase3/)
*Complete system designs for enterprise bots, medical/legal engines, e-commerce, and multi-tenant RAG architectures.*
* **Chapter 1: Internal Enterprise Systems** ([01_Internal_Enterprise_Systems.md](./Phase3/01_Internal_Enterprise_Systems.md)) — *Company wiki chatbot, Resume screening assistant, and Codebase Q&A assistant.*
* **Chapter 2: Specialized Domain Assistants** ([02_Specialized_Domain_Assistants.md](./Phase3/02_Specialized_Domain_Assistants.md)) — *Legal document assistant, Medical knowledge assistant, and Financial report analysis assistant.*
* **Chapter 3: Customer-Facing Platforms** ([03_Customer_Facing_Platforms.md](./Phase3/03_Customer_Facing_Platforms.md)) — *Customer support RAG bot, E-commerce semantic search, Research paper Q&A, and Multi-tenant SaaS platform.*

### [Phase 4: Deep RAG Engineering Questions](./Phase4/)
*Tackling production challenges: parsing tabular data, dirty PDFs, dynamic updates, access control, and latency reduction.*
* **Chapter 1: Ingestion, Chunking & Data Prep** ([01_Ingestion_Chunking_Data_Prep.md](./Phase4/01_Ingestion_Chunking_Data_Prep.md)) — *Chunk sizing, table extraction, dirty PDFs, scanned files, duplicate handling, updates/deletions, and document security (ACLs).*
* **Chapter 2: Retrieval, Generation & Optimization** ([02_Retrieval_Generation_Optimization.md](./Phase4/02_Retrieval_Generation_Optimization.md)) — *Hallucination reduction, evaluation metrics (RAGAS), latency/cost optimization, contradictory sources, citations, reranking, and query expansion.*

### [Phase 5: LLM/RAG Coding & Pseudocode](./Phase5/)
*Python-style clean implementations of end-to-end RAG, splitters, custom retrievers, rerankers, and feedback systems.*
* **Chapter 1: Ingestion & Indexing Code** ([01_Ingestion_Indexing_Code.md](./Phase5/01_Ingestion_Indexing_Code.md)) — *Basic RAG pipeline, Document loaders, Recursive Character Splitter, Embeddings generator, Batch Vector DB upserts, Vector Similarity search, and Filtered retrievers.*
* **Chapter 2: Generation, Evaluation & Ops Code** ([02_Generation_Evaluation_Ops_Code.md](./Phase5/02_Generation_Evaluation_Ops_Code.md)) — *Cross-Encoder reranking API, U-shaped prompt builder, completions generators, citation regex validators, RAG evaluation scripts, async feedback loggers, tenant security checks, and async streaming response generators.*

### [Phase 6: Interview Q&A Bank](./Phase6/)
*100 curated questions categorized from Beginner to Advanced, plus troubleshooting and production debugging.*
* **Chapter 1: Beginner & Intermediate Q&A** ([01_Beginner_Intermediate_Q&A.md](./Phase6/01_Beginner_Intermediate_Q&A.md)) — *Questions 1 to 35 covering terminology, prompting, basic RAG concepts, embedding models, and standard database interfaces.*
* **Chapter 2: Advanced & System Design Q&A** ([02_Advanced_System_Design_Q&A.md](./Phase6/02_Advanced_System_Design_Q&A.md)) — *Questions 36 to 70 covering transformer mechanics, GPU memory math, speculative decoding, and vector DB system design patterns.*
* **Chapter 3: Debugging & Production Q&A** ([03_Debugging_Production_Q&A.md](./Phase6/03_Debugging_Production_Q&A.md)) — *Questions 71 to 100 covering production debugging scenarios, performance anomalies, and deployment/cost optimizations.*

### [Phase 7: Mock Interview Test](./Phase7/)
*A 90-minute simulation with verbal, architecture, debugging, and system design mock scenarios.*
* **Chapter 1: 90-Minute Mock Interview Test** ([01_Mock_Interview_Test.md](./Phase7/01_Mock_Interview_Test.md)) — *A complete mock exam containing 31 questions (verbal, architecture, debugging, system design, tradeoffs, and final deep design) with a dedicated Answer Key at the end featuring conversational follow-ups.*

### [Phase 8: AI/ML Engineer Perspective](./Phase8/)
*Strategies for navigating screenings, mapping RAG to core ML fundamentals, and integrating tools and agents.*
* **Chapter 1: AI/ML Engineer Interview Perspective** ([01_AI_ML_Interview_Perspective.md](./Phase8/01_AI_ML_Interview_Perspective.md)) — *Recruiter screening and system design loops, conversational communication strategies (avoiding overclaiming, answering unknown implementations), and conceptual mappings (Core ML, Backend systems, Agents/Tools, and Model Context Protocol).*
* **Chapter 2: ChatGPT Voice Mode Candidate Prompt** ([02_Aman_Candidate_Voice_Prompt.md](./Phase8/02_Aman_Candidate_Voice_Prompt.md)) — *System prompt template for a ChatGPT session acting as Aman, designed for conversational response styling, resume grounding, live coding mockups, and screen-sharing simulations.*

