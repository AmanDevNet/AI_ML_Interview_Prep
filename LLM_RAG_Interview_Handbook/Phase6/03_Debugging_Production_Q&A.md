# 📓 Phase 6 Chapter 3: Debugging & Production Q&A

This chapter contains 30 debugging, troubleshooting, and production engineering questions (Questions 71 to 100) covering memory leaks, structured output failures, rate limits, and deployment optimizations.

---

## 🔵 Debugging & Troubleshooting (Q71 - Q85)

### Q71: How do you diagnose and resolve GPU Out-of-Memory (OOM) errors during the prefill phase?
* **Short Answer:** I locate the prompt batch size and sequence length parameters and implement dynamic bucket batching, activation checkpointing, or offload the KV Cache memory.
* **Detailed Explanation:** OOM during prefill occurs because processing a long prompt scales compute memory quadratically. To resolve this, we can limit the max batch size in our serving config, enable FlashAttention (to avoid allocating the $N \times N$ matrix in HBM), or implement chunked prefill (splitting the prompt prefill across multiple steps).
* **Example:** Setting `max_num_seqs` or `gpu_memory_utilization` parameters in vLLM configuration to control memory allocations.
* **Follow-up Question:** What is the difference between static and dynamic batching in LLM serving?

---

### Q72: How do you debug why an LLM fails to generate valid JSON when structured output is enabled?
* **Short Answer:** I inspect the parsing layer to confirm if we are using prompt instructions or constrained decoding, and migrate the system to grammar-based sampling constraints.
* **Detailed Explanation:** Traditional prompting (asking for JSON in text) regularly fails due to model variances. Constrained decoding (using frameworks like Outlines or Instructor) compiles the schema to a Finite State Machine and masks non-conforming tokens at the logit level during sampling, guaranteeing JSON syntax.
* **Example:** Replacing standard prompt instruction calls with `client.chat.completions.create(response_format={"type": "json_object"})` in OpenAI.
* **Follow-up Question:** How does grammar-constrained decoding handle schema validation errors?

---

### Q73: Why is your RAG bot suddenly answering "I don't know" even when the correct PDF has been uploaded?
* **Short Answer:** I debug the parsing and indexing pipeline to verify if the text was correctly extracted and check if retrieval scores fall below our safety threshold.
* **Detailed Explanation:** The failure path is often:
  1. PDF parser fails on a scanned document (outputting blank strings).
  2. Text splitter slices a key sentence at the boundary, diluting semantic similarity.
  3. The vector database similarity score drops below the minimum query threshold.
  We isolate this by printing the raw retrieved chunks for the query.
* **Example:** Checking database logs to find that the PDF was parsed as binary garbage or the chunk had a similarity score of 0.52 (below the 0.65 cutoff).
* **Follow-up Question:** How do you set up automated regression tests to prevent this failure?

---

### Q74: How do you handle and recover from LLM API Rate Limit (HTTP 429) errors in production?
* **Short Answer:** Implement exponential backoff with jitter in the API client, deploy a multi-key rotation pool, or set up local fallback models.
* **Detailed Explanation:** When receiving a 429 error, the client should wait before retrying. Using exponential backoff with random jitter prevents "thundering herd" bottlenecks. In high-traffic systems, we route requests across a pool of API keys or fall back to an open-source model hosted on a local GPU.
* **Example:** Wrapping the API call in a Tenacity decorator: `@retry(wait=wait_random_exponential(min=1, max=60), stop=stop_after_attempt(6))`.
* **Follow-up Question:** What is the difference between TPM (Tokens Per Minute) and RPM (Requests Per Minute) rate limits?

---

### Q75: How do you debug a semantic cache that is serving outdated answers to users?
* **Short Answer:** Implement an automated cache invalidation pipeline that deletes or updates cached vectors whenever their source documents are edited or deleted.
* **Detailed Explanation:** A semantic cache matches queries to past answers based on vector similarity. If document source content changes, the cached answer becomes stale. We must tag cached entries with the source document IDs and delete matching cache keys when files are updated in the ingestion pipeline.
* **Example:** Purging Redis semantic cache keys where the metadata `source_doc` matches the updated document ID.
* **Follow-up Question:** How does the choice of similarity threshold in semantic caching affect cache hit quality?

---

### Q76: Why are your retrieval queries failing to match synonym queries (e.g., matching "billing" to "invoice")?
* **Short Answer:** The embedding model was trained on out-of-domain data or has weak semantic representation, which I resolve by migrating to a stronger model or implementing hybrid search.
* **Detailed Explanation:** Basic embedding models can fail to capture domain-specific synonyms. We can resolve this by using a high-capacity model (like `text-embedding-3-large`), fine-tuning embeddings on domain QA pairs, or adding a BM25 keyword search index to capture synonym terms.
* **Example:** Migrating from a generic 384-dimensional embedding model to Cohere Multilingual Embeddings to handle domain synonyms.
* **Follow-up Question:** How do you generate synthetic training data to fine-tune embedding models?

---

### Q77: How do you debug why an LLM agent gets stuck in infinite tool-calling loops?
* **Short Answer:** I analyze the prompt logs to check if tool outputs contain errors or if the agent is failing to parse the return value, and set a hard maximum loop iteration limit.
* **Detailed Explanation:** Agents loop when the tool returns an error (e.g. `Connection Timeout`) and the model attempts to call the exact same tool with the same arguments to fix it. We resolve this by limiting the loop to $N$ runs (e.g., max 5 iterations) and structuring tool errors to instruct the model to report the failure rather than retrying.
* **Example:** Setting `max_iterations = 5` in the LangChain Agent Executor configuration.
* **Follow-up Question:** How do you handle state validation in multi-agent orchestration frameworks?

---

### Q78: Why is your search engine returning duplicate text chunks in the top-k results?
* **Short Answer:** The ingestion pipeline is splitting overlapping sections of the same text or processing identical documents in different folders, which we resolve using chunk deduplication and MMR.
* **Detailed Explanation:** Redundancy occurs when duplicate files exist in the source folders or when chunks have high overlap. We resolve this by running MD5 deduplication during ingestion and applying **Maximal Marginal Relevance (MMR)** at query time to balance document relevance with output diversity.
* **Example:** Configuring the retriever to run MMR, selecting chunks that are semantically close to the query but distant from already selected chunks.
* **Follow-up Question:** Write down the conceptual mathematical objective of MMR.

---

### Q79: How do you resolve a "Lost in the Middle" error in a long-context RAG system?
* **Short Answer:** Implement a Cross-Encoder Reranker to filter context down to only the top 3-5 chunks, and position them at the extreme beginning and end of the prompt.
* **Detailed Explanation:** When context windows are large (e.g. 32k), models ignore middle tokens. By reranking candidates and selecting only the high-precision matches, we keep prompt sizes small. If we must pass multiple documents, we sort them in a U-shape (most relevant first and last).
* **Example:** Using Cohere Rerank to reduce 50 retrieved chunks down to the 4 most relevant blocks before prompt assembly.
* **Follow-up Question:** Why do models naturally pay more attention to the beginning and end of prompts?

---

### Q80: How do you debug memory leaks in serverless GPU deployments during long-running tasks?
* **Short Answer:** Monitor the KV Cache memory allocation and implement serving engines that reuse and dynamically clear cache pages (like PagedAttention).
* **Detailed Explanation:** Memory leaks often occur because older KV Cache tensors are not deallocated from VRAM after a request terminates. We use GPU monitoring tools (like `nvidia-smi` or PyTorch memory profilers) to check allocation states and ensure serverless containers enforce request-level timeouts and clean garbage collection.
* **Example:** Setting dynamic timeouts on vLLM server processes to clear idle client contexts from VRAM.
* **Follow-up Question:** How does tensor offloading to CPU memory affect model generation latency?

---

### Q81: What is the primary cause of a high Time to First Token (TTFT) and how do you diagnose it?
* **Short Answer:** A slow prefill phase caused by processing very long prompts (compute-bound), which I diagnose by tracing execution times of the retrieval vs. generation steps.
* **Detailed Explanation:** High TTFT is caused by the model processing massive input prompts in parallel. We isolate this by logging the duration of:
  1. Embedding query generation and vector search.
  2. Context assembly.
  3. LLM prefill phase.
  We mitigate this by implementing prompt caching and reducing chunk sizes.
* **Example:** Using Langfuse to identify that vector search took 50ms, but the LLM prefill phase for a 15k token prompt took 1800ms.
* **Follow-up Question:** How does the choice of GPU hardware architecture impact prefill speed?

---

### Q82: How do you fix table formatting corruption when converting PDFs to raw text?
* **Short Answer:** Replace standard text parsers with layout-aware vision parsers that convert tables into clean Markdown or HTML table structures.
* **Detailed Explanation:** Naive parsers read characters based on coordinates, mixing columns horizontally. We run layout-aware models (like Marker or Nougat) that segment columns, detect table boundaries, and extract cells into structured Markdown grids.
* **Example:** Replacing `PyPDF2` with `unstructured[pdf]` in our ingestion parser pipeline.
* **Follow-up Question:** How do you handle tables embedded in scanned image PDFs?

---

### Q83: Why is a model ignoring the formatting guidelines (like JSON) in your prompt?
* **Short Answer:** The instructions are buried in the middle of a long prompt (attention decay), which I resolve by moving instructions to the system prompt or bottom of the user query.
* **Detailed Explanation:** Models focus on the beginning and end of prompts. If formatting instructions are placed in the middle of 10 retrieved documents, they are ignored. We resolve this by placing instructions in a dedicated System Prompt, wrapping context in distinct delimiters, and repeating formatting constraints at the very bottom of the prompt.
* **Example:** Moving *"You must output ONLY valid JSON"* from the top of the context template to the very last line of the user query.
* **Follow-up Question:** How does grammar-constrained decoding bypass this issue?

---

### Q84: How do you debug why your vector database queries are suddenly running extremely slow?
* **Short Answer:** The index size has exceeded available RAM, forcing the database to page index data from disk, which I resolve by upgrading RAM or enabling vector quantization.
* **Detailed Explanation:** Vector searches (HNSW graphs) must reside in RAM to maintain sub-10ms latency. If index sizes scale past RAM limits, disk-swapping occurs, increasing latency by 10-100x. We monitor memory usage and resolve this by enabling Scalar Quantization (converting float32 to int8 vectors) to reduce index memory footprint by 75%.
* **Example:** Upgrading a Qdrant node from 16GB to 64GB RAM when index size crossed 12GB.
* **Follow-up Question:** What is the difference between Product Quantization (PQ) and Scalar Quantization (SQ)?

---

### Q85: How do you debug prompt injection attempts in a live customer assistant bot?
* **Short Answer:** Implement input validation classification models to scan user queries and log all flagged refusals in a security dashboard.
* **Detailed Explanation:** Users try to bypass constraints using jailbreak patterns (e.g., roleplay). We intercept inputs using a fast classifier (like Llama Guard) and monitor logs for safety refusals. If a user query triggers a safety flag, we block the request before it reaches the core LLM prompt.
* **Example:** Logging a user query that contains the string "Ignore all previous instructions" and returning a standard template block.
* **Follow-up Question:** What is the performance latency cost of running input safety classification models?

---

## 🟢 Production Engineering (Q86 - Q100)

### Q86: Explain how you implement prompt compression in high-volume RAG applications.
* **Short Answer:** I run a small language model (like LLMLingua) to calculate the information entropy of retrieved chunks, removing low-perplexity filler words before prompt assembly.
* **Detailed Explanation:** Prompt compression strips redundant text. Using a small model, we calculate the perplexity of each token in the context. Tokens with low perplexity (which carry minimal unique information) are deleted, compressing the prompt by up to 50% with minimal loss in reasoning quality.
* **Example:** Compressing a 10,000-token API log file down to 4,000 high-density tokens, saving input token costs.
* **Follow-up Question:** How does prompt compression impact the accuracy of citation mapping?

---

### Q87: What is Scalar Quantization (SQ) in vector databases and when should you use it?
* **Short Answer:** SQ converts 32-bit floating-point vector dimensions to 8-bit integers, reducing database memory usage by 75% at the cost of a minor loss in retrieval precision ($<1\%$).
* **Detailed Explanation:** Floating point vectors require significant RAM (4 bytes per dimension). SQ maps the min/max value range of each dimension to an 8-bit scale (1 byte per dimension). This allows you to index 4x more vectors on the same hardware, keeping search fast and cheap.
* **Example:** Enabling SQ in Qdrant to fit 10 million 1536-dimensional vectors on a single 32GB RAM server.
* **Follow-up Question:** What is the difference between Scalar Quantization and Product Quantization?

---

### Q88: How do you design an asynchronous trace logging pipeline for production LLM systems?
* **Short Answer:** I write non-blocking trace events to a queue (like RabbitMQ or Kafka) and process them asynchronously using worker nodes, preventing logging calls from blocking user response generation.
* **Detailed Explanation:** Telemetry logging must not block inference. The API gateway generates a trace ID, collects the query, context, and response tokens, and pushes the payload to a fast message broker. Background workers pop the messages and write them to a search index database (like Elasticsearch or OpenSearch) for analysis.
* **Example:** Using an async Celery worker task in a Python FastAPI backend to log Langfuse trace payloads.
* **Follow-up Question:** How do you handle PII scrubbing inside an asynchronous logging pipeline?

---

### Q89: How do you implement a PII masking layer during document ingestion?
* **Short Answer:** I run a local Named Entity Recognition (NER) container (like Microsoft Presidio) to detect and redact sensitive data (names, emails, SSNs) before vectors are generated.
* **Detailed Explanation:** Masking PII prevents indexing sensitive data in the database. During ingestion, the raw text is scanned by an NER model. If PII is found, it is replaced with placeholder tokens (e.g. `[EMAIL_ADDRESS]`). The masked text is then sent to the embedding model and stored.
* **Example:** Redacting employee phone numbers and home addresses in HR manuals before indexing them in pgvector.
* **Follow-up Question:** How does PII masking affect vector similarity search accuracy?

---

### Q90: What is dynamic model routing and how does it optimize RAG costs?
* **Short Answer:** Dynamic routing uses a fast intent classifier to send simple queries to a cheap model and complex queries to a premium model, reducing average API costs.
* **Detailed Explanation:** Not all queries require premium models. The router (rules-based or a fast 8B model) analyzes incoming queries. Simple questions (e.g. "What is our store address?") are routed to a fast, cheap model (like GPT-4o-mini). Complex questions (e.g. "Draft a legal response...") are sent to GPT-4o.
* **Example:** Routing 70% of customer support chats to a cheap model, saving 50% in monthly API costs.
* **Follow-up Question:** How do you train or calibrate the router model to prevent routing errors?

---

### Q91: How do you structure prompt versioning and registries in production software?
* **Short Answer:** I store prompt templates as version-controlled JSON files in a dedicated repository or registry (like Langfuse or Git), decoupling prompt configurations from application code.
* **Detailed Explanation:** Hardcoding prompts in code is bad practice. We use a registry API where prompts are stored by key and version tag (e.g. `support_bot:v2`). At runtime, the application pulls the prompt template from the registry (with local caching enabled to prevent database calls), compiles the variables, and runs inference.
* **Example:** Pulling the template `customer_onboarding_v1.2` from a central Git-backed prompt database during request preprocessing.
* **Follow-up Question:** How do you run A/B tests on different prompt versions?

---

### Q92: How do you calculate cost projections for a RAG system at scale?
* **Short Answer:** I estimate costs by calculating expected query volume, average input/output tokens per query, prompt cache hit rates, and model API rates:
  $$\text{Monthly Cost} = \text{queries} \times [(\text{input\_tokens} \times \text{input\_rate}) + (\text{output\_tokens} \times \text{output\_rate})]$$
* **Detailed Explanation:** We calculate:
  - Input tokens: system instructions + history + average retrieved chunk length (e.g., 5 chunks of 500 tokens = 2500 tokens).
  - Output tokens: average model response length (e.g., 200 tokens).
  - Apply discounts for prompt caching hits.
* **Example:** Running 100,000 monthly queries on GPT-4o ($5.00/M input, $15.00/M output) with 3000 input tokens and 300 output tokens costs:
  $$100,000 \times [(\frac{3000}{1,000,000} \times 5.00) + (\frac{300}{1,000,000} \times 15.00)] = 100,000 \times [0.015 + 0.0045] = 1,950 \text{ USD per month}$$
* **Follow-up Question:** How does self-hosting open-source models change the cost calculation model?

---

### Q93: What is the performance impact of multi-GPU orchestration (tensor parallelism vs. pipeline parallelism) during inference?
* **Short Answer:** Tensor parallelism has lower latency but requires high-bandwidth connections (like NVLink); pipeline parallelism scales well across nodes but introduces latency bubbles.
* **Detailed Explanation:** In tensor parallelism, layer computations are split horizontally across GPUs, requiring all-reduce communications at every step. This makes it extremely fast but limited to a single server node. In pipeline parallelism, layers are split vertically across nodes. It allows loading massive models but introduces latency delays as one node waits for the output of the previous node.
* **Example:** Using Tensor Parallelism (TP=8) on a single HGX A100 server to serve Llama 3 70B with minimal latency.
* **Follow-up Question:** What is the role of tensor parallelism rank parameters in serving engines?

---

### Q94: How do you implement semantic search pagination?
* **Short Answer:** Retrieve a larger top-k pool (e.g., $k=100$) in a single database query, cache the results locally in the user session, and serve slices (pages) to the frontend.
* **Detailed Explanation:** Traditional SQL database offsets do not work well in vector searches because similarity scoring is relative. Instead of running a new vector search on every page click, we retrieve a larger pool (e.g., top 100 results) in the first search, store the IDs in a fast cache (like Redis), and paginate through the cached set.
* **Example:** Fetching the top 50 matches for a query, and serving pages in slices of 10 (`results[0:10]`, `results[10:20]`).
* **Follow-up Question:** What is the risk of user search intent drift during session-cached pagination?

---

### Q95: What is the trade-off of using a small local model for RAG parsing vs. an API model?
* **Short Answer:** Local models (like Llama Guard or local NER) have zero token costs and guarantee data privacy, but require dedicated GPU hardware and add hosting/maintenance overhead.
* **Detailed Explanation:** API models (like GPT-4o-mini) are easy to set up and scale instantly with zero infrastructure costs, but can be expensive at high volumes and send data outside corporate boundaries. Local models run in a secure VPC but require continuous server maintenance.
* **Example:** Using a local BERT model for PII masking to keep patient records inside a secure hospital cloud.
* **Follow-up Question:** How do you benchmark the throughput limits of local vs. remote API parsing models?

---

### Q96: Explain how to configure index parameters in an HNSW index to optimize search recall.
* **Short Answer:** Increase `ef_construction` and `M` parameters during index creation to build a denser graph, and increase `ef_search` during queries to search more paths.
* **Detailed Explanation:**
  - `M` (typically 16-64) defines the maximum number of link connections per vector node.
  - `ef_construction` (typically 64-512) controls the search depth during graph build.
  - `ef_search` (typically 16-256) defines the search buffer size during queries.
  Increasing these values improves recall but increases search latency and graph generation times.
* **Example:** Setting `ef_search = 128` in Qdrant configurations to ensure high-precision matching for medical guidelines.
* **Follow-up Question:** How does the vector dimension count affect HNSW graph RAM usage?

---

### Q97: What is prompt caching, and how do you structure prompts to maximize cache hit rates?
* **Short Answer:** Prompt caching stores precomputed KV states of static prompt prefixes. To maximize hits, place the static system prompt and files first, and put the dynamic query last.
* **Detailed Explanation:** Caching matches text prefixes from the first character. If you insert dynamic text (like a timestamp or user name) at the top of the prompt, it invalidates the prefix match for the entire prompt. We keep system templates and document context at the top and append the user query at the very end.
* **Example:**
  `[Static system instructions] + [Static policy guidelines] + [Cached block marker] + [Dynamic User Question]`
* **Follow-up Question:** How long does a prompt cache remain active in GPU memory on commercial APIs?

---

### Q98: How do you handle document formatting noise during ingestion of raw HTML pages?
* **Short Answer:** Parse pages using DOM-path selectors to strip out navigation bars, footer links, and cookie banners, keeping only the main semantic article text.
* **Detailed Explanation:** Web pages contain boilerplate. Using raw text scrapers parses navigation menu links into every chunk, diluting search precision. We write clean parsers (using libraries like BeautifulSoup or Readability) to extract only the text inside the main article container tags (like `<article>` or `id="content"`).
* **Example:** Cleaning a news site page by extracting the text from the `div.story-body` container and discarding the sidebar links.
* **Follow-up Question:** How does boilerplate text affect vector similarity scores?

---

### Q99: Design a system that handles security permissions for documents that change frequently.
* **Short Answer:** Store permissions (ACLs) as metadata keys alongside chunks, and execute updates to these keys in place in the vector database without re-generating embeddings.
* **Detailed Explanation:** Embeddings represent text semantics, which do not change when permissions update. We store ACL lists in the chunk metadata. When document permissions change in the source system, we run a metadata upsert query to update the `allowed_groups` list for all vectors matching that document ID, avoiding expensive re-embedding runs.
* **Example:** Updating a file's permission from `['admin']` to `['admin', 'marketing']` in Pinecone using a fast metadata update API call.
* **Follow-up Question:** How do you ensure that permissions updates are synced instantly to prevent data leakage?

---

### Q100: How do you design a cost monitoring system that alerts when API usage spikes?
* **Short Answer:** Integrate rate-limiters at the API gateway layer, log token counts per request to a telemetry server, and trigger alerts if daily budgets are exceeded.
* **Detailed Explanation:** We implement an API gateway (like Kong or a custom proxy) that tracks client tokens. Each request logs: `user_id`, `input_tokens`, `output_tokens`, and `cost`. We store these logs in Redis and calculate rolling costs. If a user or company exceeds their daily cost quota, we rate-limit or block their requests.
* **Example:** Setting a Slack webhook alert that triggers if the total OpenAI API bill increases by more than $100 in an hour.
* **Follow-up Question:** How do you handle fair-use pricing policies for multi-tenant SaaS clients?
