# 📓 Phase 3 Chapter 2: Specialized Domain Assistants

This chapter covers system design blueprints for high-precision, specialized domain assistants where factual correctness, precise formatting, and strict structural alignment are paramount.

---

## 📋 Designs Covered
4. [Design 4: Legal Document Assistant](#design-4-legal-document-assistant)
5. [Design 5: Medical Knowledge Assistant](#design-5-medical-knowledge-assistant)
6. [Design 6: Financial Report Analysis Assistant](#design-6-financial-report-analysis-assistant)

---

### Design 4: Legal Document Assistant

#### 1. Clarifying Questions & Constraints
* **Q:** What is the document format and average length?
  * **A:** PDFs of contracts, leases, and case law files. Average contract is 50-100 pages containing complex clauses and definitions.
* **Q:** What is the required recall precision?
  * **A:** Precision must be near 100%. Missing a single clause (e.g., a "limitation of liability" clause) is a critical system failure.
* **Q:** What is the query pattern?
  * **A:** Lawyers querying about specific clause definitions or asking the system to verify if a contract violates standard corporate guidelines.

#### 2. Architecture & Data Flow
```
[Ingestion] Contract PDF -> OCR Parser (Adobe Extract / Textract) -> Structure-Aware Chunking (by Clause/Section) -> Metadata: {clause_type, contract_parties, date} -> Domain Embedding (Legal-BERT) -> Vector DB
[Execution] Query -> Resolve legal definitions -> Extract Clause Filters -> Dense vector lookup + Keyword Search -> Reciprocal Rank Fusion (RRF) -> Reranker (Cross-Encoder fine-tuned on legal text) -> LLM (GPT-4o / Claude Opus) -> User
```

#### 3. Key Components
* **Ingestion & Processing:** We parse contracts using structure-aware parsers that detect Section headers (e.g. "Section 4.1"). Instead of token-based splitting, text is chunked strictly by clause boundaries. A definitions dictionary is built during ingestion to map defined terms (e.g., "Indemnified Parties") back to their source definition.
* **Embedding & Indexing:** We use domain-specific embedding models (like `legal-bert` or customized legal-embedding checkpoints) to capture legalese. Vectors are stored in a dedicated database with support for metadata filtering (like **Qdrant** or **Pinecone**).
* **Retrieval & Reranking:** We use a hybrid search strategy. We first run a keyword search to locate exact legal terms, combined with a vector search to catch semantic equivalents. The top 30 chunks are sent to a legal Cross-Encoder Reranker.
* **Evaluation & Monitoring:** Evaluations are run offline against a Golden Dataset of contract clauses using **groundedness** checks (does the answer draw only from the contract text?).

#### 4. Operational, Security & Privacy Concerns
* **Clause Cross-Referencing:** Legal clauses often refer to other sections (e.g., "subject to the provisions of Section 12.4"). If the retriever only fetches Section 4, it lacks the context of Section 12. We handle this during ingestion by parsing section cross-references and inserting the referred section text as an append to the parent chunk's metadata.
* **Confidentiality:** Client contracts are highly confidential. We implement strict workspace separation: each client has a dedicated database namespace to prevent cross-tenant data leakage.

#### 5. Engineering Tradeoffs
* **Clause-Based Chunking vs. Token Chunking:** Token chunking (500 tokens) is fast and simple but can cut a legal clause in half, ruining its legal interpretation. We choose clause-based chunking, which increases parsing complexity and ingestion times by 3x but guarantees that every chunk contains a complete, legally valid section.

#### 6. Final Interview-Style Answer
"To design a legal document assistant, I would implement a clause-aware ingestion pipeline and domain-specific retrieval models. During ingestion, contracts are parsed using Adobe Extract API to identify section headers and clause boundaries. We split the document strictly by clause sections rather than token counts. We also build a reference map for defined terms, embedding the definitions alongside the text chunks. We use a legal-specific embedding model and index the chunks in Qdrant. At query time, the user's question undergoes a hybrid search pass. We retrieve the top 30 matching clauses, resolve any defined terms or section cross-references dynamically by fetching their linked nodes, and run the consolidated context through a legal-trained Cross-Encoder reranker. The top 5 clauses are sent to a high-capacity model like Claude 3 Opus with a prompt template that strictly forbids the model from writing explanations that are not directly supported by the text, returning answers with exact section and page citations."

---

### Design 5: Medical Knowledge Assistant

#### 1. Clarifying Questions & Constraints
* **Q:** What are the knowledge sources?
  * **A:** Clinical practice guidelines (e.g., UpToDate articles, medical journals, drug databases like RxNorm).
* **Q:** How critical are hallucinations?
  * **A:** Life-critical. The assistant must never suggest an incorrect drug dose or make up a medical guideline.
* **Q:** How do we handle patient data?
  * **A:** The system will receive patient symptoms and history, which represents highly protected health information (PHI) under HIPAA rules.

#### 2. Architecture & Data Flow
```
[Ingestion] Medical Journal HTML/PDFs -> Clinical Text Normalizer -> Hierarchical Chunking -> Semantic Medical Ontology Tagging (UMLS/MeSH) -> Med-Embeddings (BioLinkBERT) -> Vector DB
[Execution] Patient Query -> Clinical Entity Extraction -> UMLS Ontology expansion -> Metadata Filter (e.g. Drug Class) -> Hybrid Search -> Reranker -> Strict Guardrail Layer -> LLM (with clinical-aligned prompt) -> Doctor
```

#### 3. Key Components
* **Ingestion & Processing:** XML/HTML guidelines are parsed and normalized. Chunks are split using a Hierarchical parser (Document $\rightarrow$ Section $\rightarrow$ Paragraph). A medical entity parser (like MetaMap or clinical NLP models) scans chunks to extract medical ontology codes (UMLS, MeSH, SNOMED CT) and appends them as metadata tags.
* **Embedding & Indexing:** We use domain-trained medical embedding models (like `BioLinkBERT` or `PubMedBERT`). The vectors and UMLS metadata tags are stored in an enterprise vector database (like **Milvus**).
* **Retrieval & Reranking:** When a query arrives (e.g., "treatment for pediatric asthma"), we extract clinical entities, expand them using the UMLS ontology to include synonyms (e.g., "childhood bronchospasm"), and run a hybrid metadata-restricted search. The results are reranked using a biomedical Cross-Encoder.
* **Evaluation & Monitoring:** Evaluation is performed using **Faithfulness** and **Context Precision** metrics validated by medical professionals.

#### 4. Operational, Security & Privacy Concerns
* **HIPAA Compliance:** The query orchestrator runs in a secure, HIPAA-compliant cloud environment (e.g., AWS GovCloud). Patient identifiers (names, dates of birth) are stripped at the API gateway before the query is processed by the retriever.
* **Safety Refusals:** If the system is asked a question with insufficient context, it must immediately output a standardized warning: *"Insufficient clinical data to answer. Please consult the official clinical guideline PDF linked below."*

#### 5. Engineering Tradeoffs
* **UMLS Concept Expansion vs. Direct Search:** Direct semantic search works well for general questions but can miss clinical connections (e.g., mapping a brand drug name to its generic class). Running UMLS concept expansion adds query preprocessing complexity but guarantees clinical-grade recall.

#### 6. Final Interview-Style Answer
"To design a medical knowledge assistant, I would build a HIPAA-compliant pipeline centered on medical ontology tagging and domain-specific embeddings. During ingestion, documents are parsed, split hierarchically, and scanned using clinical NLP models to append UMLS and SNOMED concept codes in the metadata. Chunks are embedded using PubMedBERT and indexed in Milvus. When a clinician queries the system, we extract the medical entities, expand them via UMLS to resolve synonyms, and execute a hybrid search using both concept filters and vector similarity. The top retrieved chunks are reranked, and passed to a clinical-aligned LLM. The prompt template restricts the model's response to only the facts present in the text, forcing the model to cite the exact medical guideline and version. Before outputting the text, a guardrail layer checks the response against a database of critical contraindications to ensure absolute patient safety."

---

### Design 6: Financial Report Analysis Assistant

#### 1. Clarifying Questions & Constraints
* **Q:** What is the input data?
  * **A:** 10-K and 10-Q SEC filings. These documents are heavy in tables, financial charts, and balance sheets.
* **Q:** What is the query pattern?
  * **A:** Calculations across years: "What was the year-over-year revenue growth of company X between 2022 and 2023?"
* **Q:** How critical is arithmetic correctness?
  * **A:** High. The LLM must not hallucinate financial calculations or mistake "billions" for "millions."

#### 2. Architecture & Data Flow
```
[Ingestion] SEC PDF/HTML -> PDFPlumber Table Extractor -> Convert tables to JSON & Markdown -> Chunk by Table Boundaries -> Metadata: {ticker, year, quarter, table_type} -> Vector DB + Relational SQL Table (for pure financial metrics)
[Execution] Query -> SQL Generator / Semantic Search Router -> Run SQL on metrics table OR Vector Search for textual footnotes -> Join Context -> Python REPL Executor (for arithmetic) -> LLM Synthesis -> Analyst
```

#### 3. Key Components
* **Ingestion & Processing:** 10-K filings are downloaded in HTML/PDF format. We use layout-aware table parsers (like `pdfplumber` or `unstructured`) to isolate tables. Tables are converted into clean Markdown formats and structured JSON objects. Textual footnotes are chunked separately.
* **Embedding & Indexing:** Textual footnotes are embedded using standard models and stored in a vector database. Tabular numbers and key financial metrics (Revenue, EBITDA) are extracted and stored in a structured **SQL database** (PostgreSQL) mapped by ticker, year, and quarter.
* **Retrieval & Reranking:** When a query involves calculations (e.g., "compare profit margins of AAPL over the last 3 years"), the system routes the request to the SQL database to fetch the exact numbers, bypassing vector search. If the query asks for qualitative explanations (e.g., "Why did revenue drop?"), the system queries the vector database for matching footnotes.
* **Evaluation & Monitoring:** Evaluations run against historical balance sheet numbers to verify arithmetic precision.

#### 4. Operational, Security & Privacy Concerns
* **Arithmetic Errors:** LLMs cannot do math reliably. We solve this by implementing a **Code Interpreter tool** (Python sandbox). The LLM is instructed to write a Python script to compute the growth rate based on the retrieved numbers, run the script, and output the result.

#### 5. Engineering Tradeoffs
* **Relational SQL Database vs. Pure Vector RAG:** Storing financial numbers as vector chunks and hoping the LLM will find them and calculate correctly is highly unreliable. We choose a **hybrid database model** (Relational SQL for numbers + Vector DB for text), which increases ingestion pipeline engineering complexity but guarantees 100% mathematical accuracy.

#### 6. Final Interview-Style Answer
"To design a financial report analysis assistant, I would implement a hybrid storage architecture: a relational database for numerical balance sheets and a vector database for textual footnotes. During ingestion, SEC filings are parsed to extract tables using PDFPlumber. The numerical values (revenue, income, assets) are written directly to structured PostgreSQL tables, while the textual footnotes and descriptions are chunked, embedded, and stored in a vector database. When an analyst asks a question, a router model determines if the query requires math. If so, it generates a precise SQL query to extract the numbers from the relational database, feeds the numbers to a Python sandbox tool to run the calculations, retrieves the output, and synthesizes the final response. If the query is qualitative, the system executes a vector search over the footnotes, reranks the matches, and presents the grounded text alongside clear links to the original SEC filing page."
