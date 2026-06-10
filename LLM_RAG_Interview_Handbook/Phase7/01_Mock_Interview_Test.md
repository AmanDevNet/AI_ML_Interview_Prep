# 📝 Phase 7 — LLM & RAG Mock Interview Simulator

This simulator is formatted as a conversational script of a real 90-minute technical interview. Each topic features the **Interviewer's Question**, the **Model Answer** (in conversational natural language), the **Interviewer's Follow-up**, and the **Follow-up Answer** inline for seamless study.

---

## 📋 Table of Contents
1. [Verbal Screening Questions (V1 - V15)](#1-verbal-screening-questions-v1---v15)
2. [Architecture Questions (A1 - A5)](#2-architecture-questions-a1---a5)
3. [Debugging Scenarios (D1 - D5)](#3-debugging-scenarios-d1---d5)
4. [System Design Questions (S1 - S3)](#4-system-design-questions-s1---s3)
5. [Tradeoff Questions (T1 - T2)](#5-tradeoff-questions-t1---t2)
6. [Final Deep Design Question (FD1)](#6-final-deep-design-question-fd1)

---

## 1. Verbal Screening Questions (V1 - V15)

### V1: Temperature Logit Scaling
* **Interviewer:** How does temperature change next-token probabilities mathematically before the sampling step?
* **Model Answer:** "Temperature division happens right before the Softmax normalization. By dividing the raw logit scores by a low temperature value, we stretch the difference between the scores, making the top token probability approach 1. If we divide by a high temperature, we flatten the differences, making the token choices more uniform and random."
* **Interviewer Follow-up:** What happens if you set the temperature parameter to exactly 0?
* **Follow-up Answer:** "Mathematically, dividing by zero is undefined. In production APIs, setting temperature to 0 is an alias that tells the inference engine to bypass probabilistic sampling entirely and perform greedy decoding, selecting the token with the highest raw logit value at every step."

---

### V2: BPE Fallback Mechanics
* **Interviewer:** How does Byte-Pair Encoding (BPE) handle misspellings or rare words it has never seen?
* **Model Answer:** "Byte-Pair Encoding builds a vocabulary of common sub-words. If it encounters a misspelled or rare word it has never seen, it breaks the word down into smaller syllables, prefixes, or individual characters that exist in its vocabulary, ensuring the model can still process it without throwing an error."
* **Interviewer Follow-up:** What is byte-level fallback, and why is it useful?
* **Follow-up Answer:** "Byte-level fallback is when the tokenizer handles characters that aren't in its vocabulary by breaking them down into raw UTF-8 bytes. This is useful because it prevents Out-of-Vocabulary errors entirely, allowing the model to process emojis or special characters without crashing."

---

### V3: DPO KL Penalty
* **Interviewer:** Why is a KL-divergence penalty necessary during DPO (Direct Preference Optimization) training?
* **Model Answer:** "The KL-divergence penalty acts as a regularizer during DPO training. It prevents the model's token distribution from drifting too far from the original SFT reference model, ensuring that the model doesn't hack the preference rewards by generating repetitive, safe, or hostile text."
* **Interviewer Follow-up:** How does the scaling factor beta control this constraint?
* **Follow-up Answer:** "Beta controls the strength of the KL penalty. A higher beta enforces a strict penalty, keeping the model aligned with the original SFT baseline, while a lower beta gives the model more freedom to optimize on the preference feedback data."

---

### V4: Cosine vs. Dot Product
* **Interviewer:** Explain when Cosine Similarity is mathematically identical to Dot Product similarity.
* **Model Answer:** "Cosine similarity divides the dot product of two vectors by the product of their magnitudes. If we normalize both embedding vectors to a length of exactly 1 before indexing, the denominator becomes 1, making cosine similarity mathematically identical to a simple dot product."
* **Interviewer Follow-up:** Why is this normalization step preferred in production indexes?
* **Follow-up Answer:** "Because computing a dot product is a single multiply-accumulate CPU/GPU instruction. Bypassing vector magnitude calculations and division operations significantly speeds up similarity search times across millions of vector rows."

---

### V5: Self-Attention Complexity
* **Interviewer:** Why does the compute complexity of self-attention scale quadratically $O(N^2)$ with sequence length?
* **Model Answer:** "Self-attention requires comparing every token in a sequence against every other token in that same sequence. If you have $N$ tokens, the model must compute an $N \times N$ attention weight matrix, which makes the compute and memory requirements scale quadratically as the prompt length increases."
* **Interviewer Follow-up:** How does FlashAttention optimize this quadratic bottleneck?
* **Follow-up Answer:** "FlashAttention doesn't change the $O(N^2)$ scaling, but it optimizes memory access. It splits the attention calculation into tiles and computes them inside the GPU's fast SRAM cache, avoiding writing the large intermediate $N \times N$ matrix to the slower GPU main memory (HBM)."

---

### V6: GQA VRAM Savings
* **Interviewer:** How does Grouped-Query Attention (GQA) reduce VRAM requirements during inference?
* **Model Answer:** "GQA groups query heads to share key and value projections. Instead of storing unique key-value states for every single attention head, multiple query heads share a single key-value group, which reduces the size of the KV cache in VRAM by up to 8x during generation."
* **Interviewer Follow-up:** Does GQA degrade the reasoning quality of the model?
* **Follow-up Answer:** "No, empirical evaluations show that GQA retains almost 99% of the accuracy of Multi-Head Attention, while providing massive VRAM savings and speed improvements."

---

### V7: SFT vs. RLHF Optimization
* **Interviewer:** What is the fundamental difference in optimization targets between SFT and RLHF?
* **Model Answer:** "SFT optimizes the model on a token-by-token basis using cross-entropy loss to match the exact characters of a gold-standard target response. RLHF optimizes the model's preference distribution using scalar rewards, training it to choose helpful and safe answers over bad ones."
* **Interviewer Follow-up:** Why is SFT performed before RLHF?
* **Follow-up Answer:** "Because RLHF is highly unstable if the model is initialized with random weights. SFT acts as a warm-up phase, teaching the model basic grammar, instructions, and formatting, so the RL phase can start with high-quality candidate generations."

---

### V8: Lost in the Middle
* **Interviewer:** Explain the "Lost in the Middle" phenomenon and its impact on prompt context layout.
* **Model Answer:** "LLMs focus their attention weights on tokens at the absolute beginning and end of long prompts, while ignoring facts placed in the middle. If you bury a key document in the middle of a massive context prompt, the model is highly likely to ignore it when writing its answer."
* **Interviewer Follow-up:** How do you structure RAG retrieval to prevent this?
* **Follow-up Answer:** "We use a Cross-Encoder reranker to filter out irrelevant chunks, keeping the prompt size small. If we must send multiple documents, we sort them in a U-shape, placing the most relevant documents at the very beginning and end of the context block."

---

### V9: Parent-Child Chunking
* **Interviewer:** How does Parent-Child (Sentence-Window) chunking improve RAG performance?
* **Model Answer:** "Parent-child chunking decouples the search index from the generation context. We split documents into small sentences (child chunks) for high-precision search, but link them in the database metadata to their parent paragraph. When a child is matched, we feed the larger parent context to the LLM."
* **Interviewer Follow-up:** Why not just search and retrieve paragraph-sized chunks directly?
* **Follow-up Answer:** "Because large paragraphs contain multiple ideas, which dilutes the embedding vector and degrades search precision. Searching small sentences is much more accurate, while returning the paragraph ensures the LLM has enough context to write an answer."

---

### V10: Reciprocal Rank Fusion
* **Interviewer:** What is Reciprocal Rank Fusion (RRF), and why is it preferred over raw score combination in hybrid search?
* **Model Answer:** "RRF is an algorithm that merges document rankings from multiple search methods by summing the reciprocal of their ranks. It is preferred because it doesn't rely on raw search scores, which have completely different scales across keyword and vector search engines."
* **Interviewer Follow-up:** What is the formula of RRF, and what does the constant $k$ represent?
* **Follow-up Answer:** "The formula is $\sum \frac{1}{k + \text{rank}(d)}$. The constant $k$ (typically set to 60) acts as a regularizer that prevents highly ranked outliers in one search method from completely dominating the final fused list."

---

### V11: Access Control (ACLs) Implementation
* **Interviewer:** How do you implement secure user-level access control (ACLs) in a vector search index?
* **Model Answer:** "We store document authorization groups directly in the metadata of each chunk vector. At query time, the orchestrator retrieves the user's groups from their JWT token and applies them as a strict pre-filtering constraint on the database search, excluding unauthorized files before vector math runs."
* **Interviewer Follow-up:** Why is filtering outputs at the prompt level a major security risk?
* **Follow-up Answer:** "Because users can use prompt injection techniques to jailbreak the model, tricking it into ignoring system rules and printing the confidential context text anyway. Security filters must be enforced at the database level before the prompt is built."

---

### V12: Prompt Caching Mechanics
* **Interviewer:** How does prompt caching function at the LLM provider API gateway level?
* **Model Answer:** "Prompt caching stores the precomputed Key-Value states of static prompt prefixes in the GPU's memory. When a new query arrives that shares the exact same prefix (like a system prompt or context manual), the engine skips the prefill compute step, returning the first token in milliseconds."
* **Interviewer Follow-up:** What is the rule of thumb for organizing a prompt to maximize cache hits?
* **Follow-up Answer:** "You must keep the prompt structured from static to dynamic. Place the system prompt and reference documents at the top, and append dynamic variables—like timestamps or the user's query—at the very end of the prompt sequence."

---

### V13: Semantic Caching Risks
* **Interviewer:** What is a semantic cache, and what is the primary risk of using it in a user-facing chatbot?
* **Model Answer:** "A semantic cache stores past user queries and generated answers in a vector database to bypass the LLM for similar queries. The risk is that the similarity match might be a false positive, returning a cached answer to a user that looks similar on paper but has a completely different meaning."
* **Interviewer Follow-up:** How do you mitigate this false-positive risk?
* **Follow-up Answer:** "We enforce a very high similarity threshold (typically $>0.96$) and restrict the semantic cache search using metadata filters, ensuring we only match queries within the same product category or tenant boundary."

---

### V14: RAG Triad Metrics
* **Interviewer:** What are the three vertices of the RAG Triad used to measure pipeline quality?
* **Model Answer:** "The RAG Triad measures context relevance, faithfulness, and answer relevance. Context relevance checks if the retriever fetched useful chunks; faithfulness checks if the generator hallucinated facts outside the context; and answer relevance checks if the model directly answered the query."
* **Interviewer Follow-up:** How do you evaluate these metrics programmatically without human reviewers?
* **Follow-up Answer:** "We use LLM-as-a-judge frameworks like Ragas. We prompt a high-capacity model (like GPT-4) to extract assertions from the response and evaluate their logical entailment against the retrieved context to calculate a mathematical grounding score."

---

### V15: Greedy Decoding Loops
* **Interviewer:** Why does greedy decoding ($T=0$) sometimes cause an LLM to fall into repetitive sentence loops?
* **Model Answer:** "Greedy decoding always selects the token with the highest probability. If a model generates a repetitive phrase, that phrase is appended to its context history. In the next step, the probability of continuing that repetitive pattern becomes the highest choice, locking the model in a loop."
* **Interviewer Follow-up:** How do you prevent these loops at inference time?
* **Follow-up Answer:** "We apply frequency and presence penalties in the sampling configuration, which subtract a scalar value from the logits of tokens that have already been generated, forcing the model to select alternative words."

---

## 2. Architecture Questions (A1 - A5)

### A1: Hybrid Search Data Flow
* **Interviewer:** Describe the data flow of a Hybrid Search engine that merges a sparse (BM25) index and a dense (vector) index.
* **Model Answer:** "In a hybrid search architecture, the user's query is sent in parallel to a sparse index (like Elasticsearch running BM25) and a dense index (like Qdrant running vector search). The sparse search returns exact keyword matches with their BM25 scores, while the dense search returns semantic matches with their cosine similarity scores. We normalize these outputs and merge them using Reciprocal Rank Fusion, which scores each document based on its relative rank in both lists. The merged list is then passed to our reranker or LLM."
* **Interviewer Follow-up:** How do you balance the weight of sparse vs. dense search in this setup?
* **Follow-up Answer:** "We can apply a linear weight parameter $\alpha$ (between 0 and 1) to adjust the scores before fusion, where $\alpha = 0.7$ favors vector search for conversational queries, and $\alpha = 0.3$ favors keyword search for part numbers or SKU codes."

---

### A2: Multi-Query & Reranker Workflow
* **Interviewer:** Describe the architecture of a Multi-Query Retrieval system that uses an LLM for query expansion before calling a reranker.
* **Model Answer:** "When a user submits a query, we first send it to a fast model (like GPT-4o-mini) to generate 3 variations of the search string. We vectorize all 4 queries and query the vector database in parallel. We consolidate all returned document lists and deduplicate them using their unique document IDs. The consolidated candidate list (e.g., 50 chunks) is then sent to a Cross-Encoder reranker, which evaluates the exact query-document attention alignment. The reranker re-scores and re-orders the chunks, and we select only the top 5 most relevant documents to build the final LLM prompt context."
* **Interviewer Follow-up:** What is the latency penalty of this multi-query and rerank step?
* **Follow-up Answer:** "The query expansion adds about 150ms and the reranker adds another 100ms. We minimize this by running the vector database lookups in parallel using async programming and hosting the reranker model on a dedicated local GPU."

---

### A3: Ingestion State Sync Flow
* **Interviewer:** Describe the ingestion state sync data flow when updating a vector index for modified files while skipping unmodified files.
* **Model Answer:** "We manage document states using a metadata sync database. During an ingestion run, we compute the MD5 hash of each source file. We compare this hash to our tracking database: if the hash matches, the file has not changed, and we skip it. If the hash is new or different, we identify the old vector IDs mapped to that document, send a delete command to clear those vectors from the database, split and embed the new document text, write the updated vectors, and save the new hashes and vector IDs to our state tracker."
* **Interviewer Follow-up:** How does this sync flow handle deletions in the source directory?
* **Follow-up Answer:** "We perform a soft-delete synchronization. If a file is missing from the source directory, we query our state database for its vector IDs, issue a batch delete command to clear those vectors from our vector store, and delete the file record from our tracking database."

---

### A4: Speculative Decoding Execution Layout
* **Interviewer:** Describe the hardware execution layout of a Speculative Decoding pipeline utilizing a draft model and a target model.
* **Model Answer:** "The system loads a small draft model (e.g., Llama 1B) and a large target model (e.g., Llama 70B) in GPU memory. The draft model runs sequentially to generate a sequence of $K$ tokens. These $K$ tokens are then fed to the target model in a single parallel prefill forward pass. The target model computes the probabilities for these tokens simultaneously: if the draft tokens match the target model's distributions, they are accepted; if a token is rejected, the target model outputs the correct token, discards the remaining draft sequence, and the draft model restarts from that new token."
* **Interviewer Follow-up:** Why does speculative decoding save latency if we are running two models?
* **Follow-up Answer:** "Because running $K$ sequential steps on a 70B model requires loading the model weights from GPU memory to cache $K$ times, which is slow. Evaluating $K$ tokens in parallel on the 70B model requires only 1 memory load step, saving significant time."

---

### A5: Logical Multi-Tenant Vector DB Isolation
* **Interviewer:** Describe a multi-tenant vector database isolation architecture using namespaces in a single vector DB cluster.
* **Model Answer:** "To build a logically isolated multi-tenant vector database, we tag every indexed vector payload with a `tenant_id` key. We configure the database to index these tags as partition keys. When a user submits a search query, the backend automatically attaches their authenticated `tenant_id` namespace to the query. The database engine limits vector similarity search strictly to that tenant's namespace partition, ensuring that vectors belonging to other companies are completely ignored."
* **Interviewer Follow-up:** How do you prevent developer bugs from accidentally querying the entire database without a namespace?
* **Follow-up Answer:** "We enforce security rules at the database connection layer: we block any search query that does not contain a valid, signed tenant namespace parameter, throwing an API permission exception before the search runs."

---

## 3. Debugging Scenarios (D1 - D5)

### D1: Prefill CUDA OOM
* **Interviewer:** During peak traffic, your serverless LLM API container throws consecutive CUDA Out-of-Memory (OOM) errors during the prefill stage. How do you resolve this?
* **Model Answer:** "CUDA OOM during prefill means the sequence length and batch size are too large for the GPU's memory. I would resolve this by checking our serving configurations (like vLLM) and reducing the maximum batch size or maximum sequence tokens per batch. I would also verify that FlashAttention is enabled to optimize attention memory, and implement chunked prefill to split long prompts across multiple steps."
* **Interviewer Follow-up:** How does the `max_num_batched_tokens` parameter affect prefill memory?
* **Follow-up Answer:** "It sets a hard limit on the total number of prompt tokens processed in a single prefill step. By lowering this limit, we force the serving engine to chunk long prompts, keeping memory usage stable at the cost of a slight increase in TTFT."

---

### D2: Legal RAG Hallucinations
* **Interviewer:** Your legal RAG bot is hallucinating sections in contract summaries. You verify that the retriever is fetching the correct PDFs, but the LLM output is incorrect. How do you debug?
* **Model Answer:** "If the retriever is fetching the correct PDFs but the model output is incorrect, it indicates a parsing or attention failure. I would print the raw text chunks passed to the LLM: if the PDF layout was parsed poorly, the text might be scrambled. If the text is clean, it means the model is ignoring the context, which I would resolve by lowering the temperature to 0, placing strict grounding instructions at the bottom of the prompt, and wrapping the context in XML tags."
* **Interviewer Follow-up:** What tool would you use to verify layout parsing quality?
* **Follow-up Answer:** "I would inspect the output of our parser using tools like pdfplumber or unstructured to compare the raw PDF visual layout with the extracted markdown strings, verifying that columns and table tables align correctly."

---

### D3: Duplicate Retrieved Paragraphs
* **Interviewer:** A customer support RAG bot is returning duplicate help paragraphs in the prompt context, wasting token space. How do you fix this?
* **Model Answer:** "Duplicate chunks occur when documents have identical sections or when chunk overlaps are too large. I would implement an exact deduplication check during ingestion using MD5 hashes. At query time, I would configure the retriever to use Maximal Marginal Relevance (MMR) search, which scores chunks based on both similarity to the query and diversity compared to already retrieved results."
* **Interviewer Follow-up:** What is a typical similarity threshold to set when filtering out duplicates?
* **Follow-up Answer:** "For exact duplicate text, we match MD5 hashes. For near-duplicates, we run Jaccard similarity or cosine distance checks and filter out any chunks that have a similarity score higher than 95%."

---

### D4: Vector DB Latency Spike
* **Interviewer:** Your vector database lookup latency spikes from 10ms to 900ms after indexing 5 million new corporate documents. What is the bottleneck, and how do you resolve it?
* **Model Answer:** "A jump in latency from 10ms to 900ms indicates that the index size has exceeded available RAM, forcing the database to page index data from disk. I would monitor the server's RAM usage and resolve this by enabling Scalar Quantization (converting float32 vectors to int8) to reduce the memory footprint by 75%, or scale out the index across a cluster of memory-optimized nodes."
* **Interviewer Follow-up:** How does HNSW graph memory usage change with vector dimensions?
* **Follow-up Answer:** "VRAM usage scales linearly with vector dimensions and link count $M$. A 1536-dimensional model requires 4x more memory than a 384-dimensional model, making dimension reduction another key tuning knob."

---

### D5: Agent Infinite Loops
* **Interviewer:** An autonomous coding agent gets stuck in an infinite loop repeatedly calling the same file reading tool helper. How do you debug and prevent this behavior?
* **Model Answer:** "Tool calling loops occur when an API returns an error or unexpected formatting, and the model attempts to call the same tool to resolve it. I would debug this by inspecting the tool execution logs. To prevent loops, I would configure a hard maximum iteration limit on the orchestrator executor, and modify the tool output wrapper so that API errors return clear instructions to report the failure rather than retrying."
* **Interviewer Follow-up:** How does setting `max_iterations = 3` help the user experience?
* **Follow-up Answer:** "It ensures the system terminates quickly and returns a graceful message (e.g., 'Unable to fetch your records at this time') rather than spinning forever, wasting token budget and leaving the user hanging."

---

## 4. System Design Questions (S1 - S3)

### S1: E-commerce Semantic Search
* **Interviewer:** Design a Semantic Search engine for an e-commerce catalog of 10 million products with a strict 50ms search latency budget.
* **Model Answer:** "To meet the 50ms latency SLA for 10 million products, we must bypass LLM generation at query time and perform hybrid retrieval. During ingestion, product descriptions are converted into 384-dimensional vectors and indexed in Qdrant, while titles and tags are indexed in Elasticsearch. At search time, the query is vectorized and sent to both databases in parallel. The results are merged using RRF, and a custom scoring algorithm boosts items based on stock status and user personalization. The final product list is validated against a Redis cache before rendering."
* **Interviewer Follow-up:** Why do we bypass the LLM generation step in this system?
* **Follow-up Answer:** "Because LLM text generation is sequential and takes seconds. E-commerce search requires sub-100ms page loads, so we use vector search only for retrieval and ranking, rendering the results directly using structured product database cards."

---

### S2: Medical Knowledge Assistant
* **Interviewer:** Design a medical guidelines search assistant that handles HIPAA-compliant clinical data and prevents dosing hallucinations.
* **Model Answer:** "My design uses a HIPAA-compliant VPC hosted on AWS GovCloud. During ingestion, medical PDFs are parsed, split hierarchically, and scanned using clinical NLP models to append UMLS concept codes in the metadata. Chunks are embedded using PubMedBERT and indexed in Milvus. At query time, patient identifiers are stripped at the API gateway before vector search. We run a hybrid search using concept filters and vector similarity, rerank the matches, and pass them to a clinical-aligned LLM. The prompt template restricts the model's response to only the facts present in the text, forcing the model to cite the exact medical guideline."
* **Interviewer Follow-up:** How do you enforce the no-hallucination dosing rule?
* **Follow-up Answer:** "We implement a post-generation validation step: an independent checker extracts all drug name and dosage patterns from the LLM output and matches them against a structured drug reference database, blocking the answer if a dosage does not align with the rules."

---

### S3: Codebase Search Assistant
* **Interviewer:** Design a codebase search assistant that parses import dependencies and inheritance paths across 1,000 repositories.
* **Model Answer:** "To search codebase structures, flat vector RAG is insufficient. I would design a Graph-RAG architecture using Neo4j. During ingestion, a GitHub webhook triggers an AST parser using `tree-sitter` to decompose files into logical functions and class nodes. We construct a dependency graph mapping import statements, class inheritances, and function calls. Each node is vectorized and indexed in Neo4j. At query time, we find the target code entity using vector similarity, and run Cypher queries to retrieve the matching code block along with all its import definitions and call paths."
* **Interviewer Follow-up:** How do you handle changes to branches and commits?
* **Follow-up Answer:** "We tag each graph node with a branch metadata key. When a commit is pushed, a Git diff script identifies the modified files, deletes their old nodes in Neo4j, and parses and indexes the updated structures."

---

## 5. Tradeoff Questions (T1 - T2)

### T1: Full Fine-Tuning vs. LoRA
* **Interviewer:** Evaluate the engineering tradeoffs of using Full Parameter Fine-Tuning versus Parameter-Efficient Fine-Tuning (PEFT/LoRA) for adapting a model.
* **Model Answer:** "Full parameter fine-tuning updates all model weights, offering maximum capability but requiring huge VRAM and GPU counts (and risking catastrophic forgetting). PEFT (like LoRA) freezes base weights and trains tiny adapter matrices, reducing GPU memory by up to 90% with almost no loss in quality. serving adapters allows dynamically hot-swapping small adapter weights on top of a single shared base model at runtime, saving hosting costs."
* **Interviewer Follow-up:** When would you choose full fine-tuning over LoRA?
* **Follow-up Answer:** "I would choose full fine-tuning when training on a massive domain shift (like a new language or complex syntax) where the low-rank updates of LoRA lack the parameter capacity to adjust the network's foundational representations."

---

### T2: Physical Isolation vs. Logical Tenant Namespaces
* **Interviewer:** Compare the tradeoffs of implementing physical database isolation (separate instances) vs. logical namespace filtering for multi-tenant SaaS platforms.
* **Model Answer:** "Physical isolation assigns separate database instances per tenant, guaranteeing 100% security and performance isolation but raising hosting costs. Logical namespaces partition data inside a shared cluster, which is highly cost-efficient but carries risk of developer bugs leaking data. We choose logical namespaces, backed by strict database security rules and automated tests."
* **Interviewer Follow-up:** How do you scale logical multi-tenancy if one tenant experiences a traffic spike?
* **Follow-up Answer:** "We implement rate-limiting at the API gateway layer per tenant ID, and use vector databases that support partition-level resource limits to prevent a single tenant from starving resources from other clients."

---

## 6. Final Deep Design Question (FD1)

### FD1: Enterprise Customer Support RAG System
* **Interviewer:** Design an enterprise Customer Support RAG system that automates ticket replies, triggers order cancellations via APIs, and synchronizes document indices when support articles change in real-time.
* **Model Answer:** "To design this customer support RAG system, I would build a low-latency orchestrator that runs user profile lookups, vector searches, and intent classification in parallel. During ingestion, help center articles are chunked into 256-token nodes and indexed in Qdrant. At query time, we retrieve the customer's CRM data from Redis while querying the vector database for matching support articles. The orchestrator checks if the user's query requires action (like refund processing). If so, the LLM initiates tool calling, generating structured JSON arguments which are validated and executed by our backend APIs. If it's a general question, the context is combined with the user's profile and sent to a fast model like GPT-4o-mini. The entire system is monitored using LangSmith to track deflection rates and ensure safety refusal compliance."
* **Interviewer Follow-up:** How do you ensure the help center article updates are synced in real-time without interrupting active chats?
* **Follow-up Answer:** "We use a Blue-Green indexing strategy: when help articles are updated, we index the new vectors in a background collection. Once the ingestion run is complete, we dynamically update the orchestrator's collection pointer, ensuring seamless transitions without search downtime."
