# 📗 Phase 2 Chapter 4: Advanced Retrieval

This chapter covers how to optimize retrieval quality in production systems by combining dense semantic vectors, sparse keyword indices, reranker models, and query expansion techniques.

---

## 📋 Topics Covered
20. [Topic 20: Hybrid Search](#topic-20-hybrid-search)
21. [Topic 21: Keyword Search vs. Semantic Search](#topic-21-keyword-search-vs-semantic-search)
22. [Topic 22: BM25](#topic-22-bm25)
23. [Topic 23: Reranking](#topic-23-reranking)
24. [Topic 24: Query Rewriting](#topic-24-query-rewriting)
25. [Topic 25: Multi-Query Retrieval](#topic-25-multi-query-retrieval)

---

### Topic 20: Hybrid Search

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Hybrid Search is the retrieval strategy that combines sparse keyword-based search (like BM25) and dense semantic vector search (like cosine distance on embeddings) into a single query pass, merging the results using a fusion algorithm.
* **Why it matters:** Dense vector search is excellent at conceptual matching but struggles with exact part numbers, product SKUs, unique IDs, or rare acronyms. Sparse search matches exact terms with high precision. Hybrid search combines both strengths.
* **How it affects answer quality:** It increases context precision and recall. If a user asks "How do I fix error error-code-1294?", hybrid search matches the exact error code via sparse search and matches the semantic troubleshooting steps via dense search.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Complexity/Latency vs. Coverage. Implementing hybrid search requires running two separate database indexes and merging their output scores, which increases storage costs and query latency.
* **Production Example:** A search page on an e-commerce site. The keyword index matches the exact model name ("iPhone 15 Pro Max"), while the vector index matches the semantic category queries ("best phone for video recording").

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Hybrid search merges sparse keyword search and dense vector search. This matches both exact codes/product IDs and conceptual meanings, providing a much higher retrieval recall than either method alone."
* **Top Q&As:**
  * **Question 1:** What is Reciprocal Rank Fusion (RRF), and how does it merge sparse and dense results?
    * **Answer:** RRF is a model-free rank fusion algorithm. It scores each document based on its relative rank in the sparse and dense result lists rather than their raw distance scores (which have different scales). The RRF score of a document $d$ is:
      $$\text{RRF\_Score}(d \in D) = \sum_{m \in M} \frac{1}{k + r_m(d)}$$
      where $M$ represents the search methods (sparse and dense), $r_m(d)$ is the rank of document $d$ in method $m$, and $k$ is a constant (typically 60) that regularizes the impact of low-ranked items.
    * **Follow-up:** Why is RRF preferred over weighted linear combination? (Answer: Because RRF does not require calibrating or normalizing raw vector distance scores and keyword score scales, making it highly stable across different embedding models and database updates).

---

### Topic 21: Keyword Search vs. Semantic Search

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Keyword Search matches exact word characters using inverted indices. Semantic Search matches conceptual meaning by computing distance between dense vectors in multi-dimensional space.
* **Why it matters:** Relying on only one approach creates blind spots. Keyword search misses synonyms (e.g., searching "automobile" misses "car"). Semantic search can miss exact strings (e.g., searching a specific UUID).
* **How it affects answer quality:** Balancing the two ensures the retriever retrieves both the exact entities mentioned in the query and the surrounding conceptual context.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:**
  - **Keyword:** Extremely cheap to store and compute, fast, exact, but lacks conceptual understanding.
  - **Semantic:** Captures synonyms and multilingual relationships, but requires GPU compute for embeddings and expensive memory to store vector graphs.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Keyword search is an exact string matching process using inverted indexes, while semantic search uses deep learning embeddings to match concepts based on their position in high-dimensional vector space."
* **Top Q&As:**
  * **Question 1:** Give a scenario where semantic search performs worse than keyword search.
    * **Answer:** When looking for specific product models, serial numbers, log codes (e.g., "CVE-2024-3814"), or specific names. The embedding model will often map these rare tokens to general category concepts, failing to retrieve the exact document containing the unique identifier, which keyword search would locate instantly.
    * **Follow-up:** How do we handle this in system design? (Answer: We deploy a hybrid search engine where the database indexes both the raw text fields via BM25 and the dense vector representations).

---

### Topic 22: BM25

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** BM25 (Best Matching 25) is a sparse search ranking algorithm that estimates the relevance of a document to a query based on term frequency (TF), inverse document frequency (IDF), and document length normalization.
* **Why it matters:** It is the industry standard for keyword search, outperforming basic TF-IDF by preventing single-word repetitions from artificially inflating a document's relevance.
* **How it works:**
  1. It calculates TF (how often the term appears in the document) but applies a saturation limit: as a term repeats more, the incremental score gain decreases.
  2. It calculates IDF (favoring rare query terms over common words like "the").
  3. It applies document length normalization: longer documents are penalized because they have a higher probability of containing words by chance.
* **Simple Analogy:** TF-IDF is like counting occurrences. BM25 is like grading a paper: if you write the word "neural" 50 times in a 1-page essay, BM25 knows you aren't 50 times smarter; it caps the value of repetition and adjusts for essay length.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Configuring Elasticsearch, OpenSearch, or pg_trgm in PostgreSQL to index text logs and run fast, normalized keyword searches.
* **Engineering Tradeoffs:** Tuning hyperparameters $k_1$ and $b$:
  - $k_1$ controls term frequency saturation.
  - $b$ controls document length normalization.
  Tuning these parameters requires domain knowledge of the average document length and search characteristics.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "BM25 is a keyword ranking algorithm that improves upon TF-IDF by adding term frequency saturation and document length normalization, ensuring short, dense matches are ranked higher than long, repetitive documents."
* **Top Q&As:**
  * **Question 1:** Write down the conceptual formula of BM25 and explain its parameters.
    * **Answer:** The score of a document $D$ for a query $Q$ with terms $q_i$ is:
      $$\text{Score}(D, Q) = \sum_{i=1}^n \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot \left(1 - b + b \cdot \frac{|D|}{\text{avgdl}}\right)}$$
      where $f(q_i, D)$ is the term frequency in the document, $|D|$ is the document length, and $\text{avgdl}$ is the average document length in the corpus. Parameter $k_1$ (typically 1.2 to 2.0) limits term frequency saturation, and $b$ (typically 0.75) controls the scaling of the document length penalty.
    * **Follow-up:** What happens to the score as $b$ approaches 0? (Answer: The length penalty is completely disabled, meaning long documents are not penalized for containing more words).

---

### Topic 23: Reranking

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Reranking is a two-stage retrieval process where a fast, lightweight retriever (dense or hybrid) fetches an initial candidate pool (e.g., $k=100$), and a computationally heavy Cross-Encoder model evaluates and re-orders those candidates to output a high-precision subset (e.g., $k=5$).
* **Why it exists:** Dense embedding retrievers evaluate query and documents independently (Bi-Encoder), which can miss complex semantic relationships. Cross-Encoders evaluate the query and document together, but are too slow to run across the entire database. Rerankers act as a second-stage filter to get the precision of Cross-Encoders at scale.
* **How it affects answer quality:** High. It acts as an attention filter, sorting the absolute most relevant information to the very top of the context window, directly reducing "Lost in the Middle" errors and generator hallucinations.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Integrating a Cohere Rerank or BGE-Reranker API into the retrieval pipeline after vector search, shifting the most relevant paragraph from rank 18 to rank 1.
* **Engineering Tradeoffs:** Precision vs. Latency. Running a reranker adds a second model inference step, which can add 50-200ms of latency to the query pipeline. However, it allows you to feed far fewer, higher-quality tokens to the generator, which can save overall latency and generation cost.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Reranking uses a high-precision Cross-Encoder model to re-evaluate and re-sort a candidate pool of documents fetched by an initial fast search. It puts the absolute most relevant facts at the top of the prompt context, maximizing generation quality."
* **Top Q&As:**
  * **Question 1:** Why is a Cross-Encoder so much more accurate than a Bi-Encoder, but too slow for first-stage search?
    * **Answer:** A Bi-Encoder compresses text into a single vector, losing token-to-token comparison data. A Cross-Encoder processes the query and document tokens together, calculating self-attention across all query and document tokens simultaneously. This allows the model to capture deep semantic relationships, but it scales quadratically, making it too slow to run across millions of database documents.
    * **Follow-up:** How do you optimize reranker latency in production? (Answer: We limit the candidate pool size (e.g., only reranking the top 20 documents instead of the top 100), host the reranker model on a dedicated, low-latency GPU node, and run execution in parallel with prompt template loading).

---

### Topic 24: Query Rewriting

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Query Rewriting is the preprocessing step where a lightweight LLM reformulates the raw user input into one or more optimized search queries to improve retrieval matching.
* **Why it exists:** Users write messy queries. They include conversational filler ("Can you tell me how to..."), write typos, or refer to conversational history (e.g., "How do I fix *it*?"). Query rewriting normalizes the search query before running vector lookup.
* **How it works:**
  1. The user's query and the chat history are sent to a fast model (like Llama 3 8B or GPT-3.5).
  2. The model resolves pronouns (e.g., converting "it" to "the database connection error").
  3. It outputs a clean, search-optimized string, removing conversational noise.
  4. The optimized string is sent to the search database.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** In a multi-turn chat, a user asks "What is our vacation policy?" and follow up with "What about sick leave?". Query rewriting reformulates the second query to "What is our corporate policy on sick leave?" to ensure correct retrieval.
* **Engineering Tradeoffs:** Accuracy vs. Latency/Cost. Query rewriting adds a preprocessing LLM call, which increases query costs and adds 100-300ms of latency before the retrieval step even begins.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Query rewriting uses a fast LLM to translate conversational, pronoun-heavy user queries into clean, search-optimized keywords. This ensures the retriever queries the database with the full context of the conversation."
* **Top Q&As:**
  * **Question 1:** How do you write a prompt for a query rewriting model to prevent it from hallucinating details?
    * **Answer:** We write a strict system prompt containing few-shot examples of conversational history mapped to rewritten queries, and instruct the model: *"You must only output the optimized search query. Do not add explanations, do not answer the question, and do not introduce any new facts not present in the chat history."*
    * **Follow-up:** How do you handle fallback if the query rewriter output is bad? (Answer: We run a parallel query where we search using both the original user query and the rewritten query, merging the results using RRF to ensure we have a fallback search path).

---

### Topic 25: Multi-Query Retrieval

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Multi-Query Retrieval (or Query Expansion) is a strategy where an LLM generates multiple distinct variations of the user's query from different perspectives, retrieves documents for each variation, and merges the results.
* **Why it exists:** A user's query might represent a narrow perspective. Generating multiple variations (e.g., looking for synonyms or related sub-topics) expands the search coverage, capturing relevant documents that would have been missed by a single vector search query.
* **How it works:**
  1. Prompt a fast LLM: *"Generate 3 different search queries to retrieve documents for: [User Query]"*.
  2. The LLM outputs 3 distinct queries (e.g., different wording, synonyms).
  3. Run vector search on the database for all 3 queries plus the original query.
  4. Deduplicate the retrieved document lists.
  5. Pass the consolidated document pool to the reranker or generator.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** User queries: "How do we scale our database?". The model generates: "Database partitioning guide", "horizontal scaling Postgres", and "sharding strategies". This retrieves different documents covering all three technical solutions.
* **Engineering Tradeoffs:** Recall vs. Latency/Database Load. Multi-query retrieval maximizes retrieval coverage (high recall), but multiplies the database query load by 4x, which can bottleneck database read throughput and increase costs.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Multi-query retrieval uses an LLM to generate multiple variations of the user's prompt. We run search queries for all variations, deduplicate the results, and merge them, maximizing our chances of pulling the necessary background documents."
* **Top Q&As:**
  * **Question 1:** How do you merge the results of multiple vector search queries in a Multi-Query setup without blowing up the context window?
    * **Answer:** We retrieve the top 10-20 documents for each generated query. We consolidate these lists into a single set, remove duplicates based on document ID, and run the unique set through a Reranker model. The reranker filters the consolidated set down to only the top 5 most relevant chunks to prevent context overflow.
    * **Follow-up:** How do you optimize execution speed? (Answer: We execute all the database vector search queries in parallel using asynchronous programming (e.g., Python's `asyncio`), ensuring that the database query phase takes only as long as a single slow lookup).
