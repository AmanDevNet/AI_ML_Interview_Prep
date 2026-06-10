# 📓 Phase 3 Chapter 1: Internal Enterprise Systems

This chapter covers system design blueprints for internal corporate tools and developer productivity systems. 

---

## 📋 Designs Covered
1. [Design 1: Company Documentation Chatbot](#design-1-company-documentation-chatbot)
2. [Design 2: Resume Screening Assistant](#design-2-resume-screening-assistant)
3. [Design 3: Codebase Q&A Assistant](#design-3-codebase-qa-assistant)

---

### Design 1: Company Documentation Chatbot

#### 1. Clarifying Questions & Constraints
* **Q:** What is the document volume and format?
  * **A:** ~100,000 documents across Confluence pages, PDF manuals, and Slack logs. Average file size is 5 pages.
* **Q:** What is the user scale and latency budget?
  * **A:** 10,000 internal employees. Max concurrent users ~500. Latency budget: TTFT < 500ms, total generation time < 3 seconds.
* **Q:** Are there data privacy constraints?
  * **A:** Yes, strict user-level document access control (Active Directory integration). Users must never retrieve content they don't have permissions to read.

#### 2. Architecture & Data Flow
```
[Ingestion] Confluence Webhook / S3 Push -> PyPDF/Markdown Parser -> Recursive Chunking (500 tokens) -> metadata: {doc_id, acl_groups, path} -> text-embedding-3-small -> Vector DB (Qdrant with ACL metadata filtering)
[Execution] User Query -> AD Authorization Lookup -> Construct ACL filter query -> Vector Search + BM25 Hybrid -> Reciprocal Rank Fusion (RRF) -> Reranker (BGE-Reranker) -> Top 5 Chunks -> LLM (GPT-4o-mini) -> User
```

#### 3. Key Components
* **Ingestion & Processing:** Incremental crawler utilizing Confluence API webhooks and S3 event triggers. Text is parsed using vision-based parsers to preserve layout hierarchy. Chunks are split using a Recursive Character Spitter (512 token limit, 50 token overlap).
* **Embedding & Indexing:** OpenAI `text-embedding-3-small` (1536 dimensions). Indexed in **Qdrant** using an HNSW graph configuration with scalar quantization enabled to fit the index entirely in memory.
* **Retrieval & Reranking:** Hybrid search (vector cosine similarity + BM25 keyword matching) to catch exact acronyms. Results are fused using RRF. The top 50 matches are sent to `bge-reranker-large` on a local GPU node, returning the top 5 to the LLM.
* **Evaluation & Monitoring:** Automated offline checks using a synthetic evaluation suite of 500 questions scored by Ragas. Live telemetry logged using OpenTelemetry and Langfuse to monitor latency and rate limits.

#### 4. Operational, Security & Privacy Concerns
* **Document-Level ACLs:** Every chunk metadata payload contains an array: `allowed_groups: ['engineering', 'hr-admins']`. At query time, the orchestrator retrieves the user's Active Directory groups from the session token and passes them as a strict pre-filter.
* **Stale Context:** Document updates trigger Confluence event pushes. The ingestion service updates the corresponding vectors in place using the document ID as a primary key, deleting old chunk vectors before indexing the new version.

#### 5. Engineering Tradeoffs
* **HNSW Index vs. RAM Usage:** HNSW provides sub-10ms search times but requires keeping the entire graph in VRAM/RAM. We trade a fraction of precision (1-2%) to enable **Scalar Quantization (float32 to int8)**, reducing vector memory overhead by 75% and saving hardware costs.

#### 6. Final Interview-Style Answer
"To design this company documentation chatbot, I would build a hybrid search architecture with strict pre-retrieval access control filtering. During ingestion, documents are parsed using layout-aware engines, split recursively into 500-token chunks, and tagged in the metadata with their respective Active Directory security groups. The vectors are stored in a Qdrant database using an HNSW index with scalar quantization to control RAM costs. When a user queries the bot, the backend retrieves their Active Directory credentials and applies them as a metadata filter during a hybrid BM25 and vector search pass. The top 50 results are ranked using a Cross-Encoder rerank model to pull the top 5 most relevant, distinct chunks, which are placed in a U-shape within the prompt template of a fast model like GPT-4o-mini. Telemetry is tracked using Langfuse to continuously evaluate context precision and detect any document permission drift."

---

### Design 2: Resume Screening Assistant

#### 1. Clarifying Questions & Constraints
* **Q:** What is the document format and volume?
  * **A:** ~50,000 resume PDFs and DOCX files.
* **Q:** What is the query pattern?
  * **A:** HR recruiters pasting a long job description (JD) and asking to retrieve the top 10 matching resumes, or comparing a specific resume to a JD.
* **Q:** What is the privacy constraint?
  * **A:** Resumes contain PII (Names, phone numbers, addresses). PII must be masked before sending data to external APIs.

#### 2. Architecture & Data Flow
```
[Ingestion] Resume Upload -> PDF/DOCX Parser -> Microsoft Presidio (PII Masking) -> Key-Value Metadata Extraction (skills, experience, degree) -> Semantic Paragraph Chunking -> Embeddings -> Vector DB
[Execution] Recruiter J.D. Input -> Extract Required Skills Profile (via LLM schema) -> Run Metadata Structured Filters (e.g. Min 5 yrs experience) -> Dense Vector Search -> Cohere Rerank -> LLM Match Explanation -> Recruiter
```

#### 3. Key Components
* **Ingestion & Processing:** Document parser (Unstructured) extracts text. A local PII scrubber (Microsoft Presidio) detects and masks names, emails, and phone numbers with placeholders (e.g., `[REDACTED_NAME]`). An extraction LLM runs once to extract structured key metrics (years of experience, degree level, key skills) and writes them to a metadata dictionary.
* **Embedding & Indexing:** We use a model optimized for matching short, dense profiles (like `bge-large-en-v1.5`). The index is stored in a relational vector database (like **pgvector** in PostgreSQL), which is ideal because the recruiter queries are heavily dependent on mixed SQL filters (e.g., matching years of experience and city locations) combined with semantic text matching.
* **Retrieval & Reranking:** We use metadata filters first (e.g., matching degree requirement), then run cosine similarity on the resume text vectors. The top 20 candidates are reranked based on alignment with the job description.
* **Evaluation & Monitoring:** Custom validation metrics measuring **Precision@K** (how many of the retrieved top-k resumes were graded as a match by recruiters).

#### 4. Operational, Security & Privacy Concerns
* **PII Leakage:** Running PII masking *prior* to vector indexing. If unmasked names are indexed, queries like "Find Bob Smith's resume" would work, which violates employee privacy guidelines.
* **Algorithmic Bias:** Embeddings can inherit gender or ethnic bias from resume text (e.g., identifying gendered pronouns or activities). We mitigate this by stripping out non-professional hobbies and demographic metadata during the text cleaning phase.

#### 5. Engineering Tradeoffs
* **JSON Metadata Filtering vs. Strict SQL Columns:** Storing structured attributes as JSON fields in PostgreSQL pgvector is highly flexible for different job types. However, if search volumes scale, we trade flexibility for performance by writing dedicated column indexes (like B-Trees) on high-frequency search fields like `years_of_experience` and `location` to prevent database bottlenecks.

#### 6. Final Interview-Style Answer
"For the resume screening assistant, the priority is data privacy and highly structured filtering. My design uses a layout-aware PDF parser followed by a local Microsoft Presidio container to mask PII (names, phone numbers, addresses) before vectorization. I will run a structured extraction pass using a lightweight LLM to pull key metrics like years of experience and education level into metadata fields. The text and metadata are stored in a PostgreSQL database using pgvector. When a recruiter inputs a job description, we first parse the JD to extract structural filters, apply these as hard SQL conditions on the database, and run a vector similarity search on the remaining resumes using `bge-large-en-v1.5` embeddings. The top 20 matches are reranked using a Cross-Encoder, and the final top 10 are presented to the recruiter alongside a generated explanation summarizing why their skills match the JD, ensuring complete auditability and preventing demographic bias."

---

### Design 3: Codebase Q&A Assistant

#### 1. Clarifying Questions & Constraints
* **Q:** What is the codebase size?
  * **A:** ~1,000 repositories, average repository size is 2,000 files.
* **Q:** How do we handle code dependencies?
  * **A:** Code contains import relationships. A query like "Where is this class defined?" requires syntax-aware graph matching, not just text matching.
* **Q:** How often does the code change?
  * **A:** Real-time push events on GitHub.

#### 2. Architecture & Data Flow
```
[Ingestion] GitHub Webhook -> Git Clone -> AST Parser (tree-sitter) -> Extract Code Nodes (Functions, Classes) & Import Graph -> Write Graph DB (Neo4j) -> Generate Node Embeddings -> Index in Neo4j Vector Index
[Execution] Developer Query -> Intent Parser -> Resolve Entity references -> Graph Query (cypher) + Vector Path Traversals -> Compile Context Map -> LLM (Claude 3.5 Sonnet) -> Developer
```

#### 3. Key Components
* **Ingestion & Processing:** Abstract Syntax Tree (AST) parser (using `tree-sitter`) parses code files. Instead of splitting by lines, code is chunked by function blocks, class definitions, and module scopes. The parser maps the imports to construct a dependency graph (Node A imports Node B).
* **Embedding & Indexing:** We use code-specific embedding models (like `text-embedding-ada-002` or `bge-large-en-v1.5` fine-tuned on code). The data is indexed in **Neo4j**, which serves as both a Graph Database (mapping code dependencies) and a Vector Store (indexing code semantics).
* **Retrieval & Reranking:** We resolve code entities using graph traversals. If the user asks about class `UserManager`, the retriever finds the node `UserManager` in the graph database, traces its outgoing import edges, and retrieves the definitions of all imported helper classes, building a contextually complete map of dependencies.
* **Evaluation & Monitoring:** Standard programming QA evaluation using AST validating compilers to check if the generated code snippets have valid syntax.

#### 4. Operational, Security & Privacy Concerns
* **State Sync:** Code branches shift continuously. We restrict the search index to the active branch (e.g. `main` or the user's active PR branch) by tagging each node with a `branch` metadata key and running clean Git diff updates to only re-embed edited files.

#### 5. Engineering Tradeoffs
* **Graph-RAG vs. Flat Vector RAG:** Flat vector search on code files is fast but fails on structural questions like "Find all files that call this deprecated function." Graph-RAG (Neo4j + vectors) solves this structural search perfectly but increases database query latency and ingestion parsing complexity by 5x.

#### 6. Final Interview-Style Answer
"To build a codebase Q&A assistant, a basic flat vector search is insufficient because it misses the logical structure of code. I would design a Graph-RAG architecture using Neo4j and AST parsers. During ingestion, a GitHub webhook triggers an AST parser using `tree-sitter` to decompose files into logical functions and class nodes. We construct a code dependency graph mapping import statements, class inheritances, and function calls. Each function node is vectorized and indexed within Neo4j. When a developer asks a question, we parse their query to identify key code entities, resolve these entities to graph nodes, and run graph traversals (using Cypher queries) to retrieve the target class and all its dependency definitions. This complete code context is formatted into a prompt template and passed to Claude 3.5 Sonnet—which is optimized for code reasoning—to write the final answer with syntactically valid code blocks."
