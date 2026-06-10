# 📓 Phase 4 Chapter 2: Retrieval, Generation & Optimization

This chapter contains deep, production-level answers to critical engineering challenges in search optimization, quality metrics, latency reduction, cost control, and response synthesis.

---

## 📋 Questions Covered
11. [Q11: How do you prevent hallucination?](#q11-how-do-you-prevent-hallucination)
12. [Q12: How do you evaluate retrieved context?](#q12-how-do-you-evaluate-retrieved-context)
13. [Q13: How do you evaluate final answers?](#q13-how-do-you-evaluate-final-answers)
14. [Q14: How do you reduce latency?](#q14-how-do-you-reduce-latency)
15. [Q15: How do you reduce cost?](#q15-how-do-you-reduce-cost)
16. [Q16: How do you handle long documents?](#q16-how-do-you-handle-long-documents)
17. [Q17: How do you handle contradictory sources?](#q17-how-do-you-handle-contradictory-sources)
18. [Q18: How do you add citations?](#q18-how-do-you-add-citations)
19. [Q19: How do you detect bad retrieval?](#q19-how-do-you-detect-bad-retrieval)
20. [Q20: How do you improve retrieval quality?](#q20-how-do-you-improve-retrieval-quality)
21. [Q21: What is reranking and when should you use it?](#q21-what-is-reranking-and-when-should-you-use-it)
22. [Q22: What is hybrid search?](#q22-what-is-hybrid-search)
23. [Q23: What is query expansion?](#q23-what-is-query-expansion)
24. [Q24: What is query rewriting?](#q24-what-is-query-rewriting)
25. [Q25: What is metadata filtering?](#q25-what-is-metadata-filtering)

---

### Q11: How do you prevent hallucination?

* **Definition:** Hallucination prevention refers to system-level constraints and prompt configurations that restrict the LLM to generating answers based only on the provided context, eliminating speculative statements.
* **Practical Solution:**
  1. Set the model's sampling **Temperature to 0** to enforce deterministic decoding.
  2. Write an explicit instruction template: *"You must answer the user query based ONLY on the context provided in the `<context>` XML tags. If the answer cannot be found in the context, you must state: 'I don't know'."*
  3. Run a post-generation **Natural Language Inference (NLI)** alignment check to verify that all claims in the generated response are logically entailed by the retrieved context.
* **Tradeoff:** Minimizing hallucinations completely makes the model less conversational and can result in false-positive refusals if the answer requires minor synthesis of the retrieved text.
* **Interview-Ready Answer:** "I prevent hallucinations by setting the inference temperature to 0, framing the prompt with a strict grounding constraint and an explicit refusal clause, and running an automated post-generation NLI check to verify that every claim in the response is logically supported by the retrieved context."

---

### Q12: How do you evaluate retrieved context?

* **Definition:** Context evaluation is the calculation of quantitative metrics to evaluate the relevance, completeness, and noise level of the text chunks retrieved from the database.
* **Practical Solution:** I use the **Ragas** framework to measure two primary retrieval metrics:
  1. **Context Precision:** The ratio of retrieved chunks that are actually relevant to the query, verifying that the top-ranked documents contain the target answers.
  2. **Context Recall:** Measures if all key facts needed to answer the question were successfully retrieved from the database.
* **Tradeoff:** Evaluating context requires using a high-capacity model (like GPT-4) as an evaluation judge, which adds API cost and latency during offline testing runs.
* **Interview-Ready Answer:** "I evaluate retrieved context using Context Precision and Context Recall metrics from the Ragas framework. This measures if the database returned all the facts needed to answer the query, and checks if the most relevant information is correctly ranked at the top of the context window."

---

### Q13: How do you evaluate final answers?

* **Definition:** Final answer evaluation measures the factual correctness and relevance of the LLM's generated response in relation to both the user's query and the retrieved documents.
* **Practical Solution:** I implement automated evaluation using two key metrics:
  1. **Faithfulness (Groundedness):** We prompt a judge LLM to extract all claims from the generated answer and check if they are supported by the retrieved context.
  2. **Answer Relevance:** Evaluates if the response directly addresses the user's query (measuring semantic alignment).
  3. **Semantic Similarity:** Computes cosine similarity between the generated answer and a human-verified target answer in our golden dataset.
* **Tradeoff:** Automated evaluation is highly scalable but can occasionally inherit biases from the judge model, requiring periodic human validation.
* **Interview-Ready Answer:** "I evaluate final answers by measuring Faithfulness and Answer Relevance. Faithfulness verifies that every claim in the generated text is supported by the retrieved context to ensure there are no hallucinations, while Answer Relevance checks if the model directly answered the user's question."

---

### Q14: How do you reduce latency?

* **Definition:** Latency reduction refers to optimizations designed to speed up the end-to-end RAG pipeline, focusing on Time to First Token (TTFT) and Inter-Token Latency (ITL).
* **Practical Solution:**
  1. **Parallelization:** Run vector search and user profile database lookups concurrently using asynchronous programming (`asyncio`).
  2. **Prompt Caching:** Place static system prompts and templates at the beginning of the query to enable prompt cache hits, cutting prefill compute.
  3. **Quantization:** Host open-source models in INT8/INT4 precision using optimized engines like **vLLM** to speed up decoding rates.
  4. **No Reranker on Simple Queries:** Run a fast intent classifier to bypass the reranker step for simple queries.
* **Tradeoff:** Bypassing the reranker saves 100-200ms of latency but can slightly reduce retrieval precision on complex queries.
* **Interview-Ready Answer:** "I reduce latency by running database lookups in parallel, leveraging prompt caching to speed up the prefill phase, and serving quantized models through vLLM to optimize token generation. For latency-critical apps, I bypass the reranking step on simple queries using a fast intent classifier."

---

### Q15: How do you reduce cost?

* **Definition:** Cost reduction is the optimization of API token consumption and GPU compute utilization across the ingestion and execution phases.
* **Practical Solution:**
  1. **Semantic Caching:** Store previous query-response pairs in a Redis vector cache. If a new query matches a cached entry with high similarity ($>0.96$), return the cached response directly, bypassing the LLM.
  2. **Model Routing:** Route simple queries to a cheap model (e.g., GPT-4o-mini) and send only complex reasoning queries to a premium model (e.g., GPT-4o).
  3. **Prompt Compression:** Strip out filler words and formatting noise from retrieved chunks before building the prompt.
* **Tradeoff:** Semantic caching significantly reduces costs but can serve outdated or slightly mismatched answers if the similarity threshold is set too low.
* **Interview-Ready Answer:** "I reduce costs by implementing semantic caching with Redis to bypass LLM calls for repeated queries, routing simple classifications to smaller, cheaper models, and compressing retrieved context chunks to minimize input token volume."

---

### Q16: How do you handle long documents?

* **Definition:** Long document handling is the optimization required to index and retrieve information from documents that exceed typical chunk sizes (e.g., 100-page manuals).
* **Practical Solution:**
  1. **Hierarchical Ingestion:** Parse the document into a tree structure: Document $\rightarrow$ Chapter $\rightarrow$ Section $\rightarrow$ Paragraph.
  2. **Summary Embedding:** Generate a text summary for each chapter. Embed the summaries as search entry points.
  3. **Retrieve and Expand:** When a chapter summary matches a query, fetch the specific paragraphs within that chapter, preventing the need to load the entire document into the prompt.
* **Tradeoff:** Hierarchical chunking preserves structural context and keeps prompt sizes small, but requires complex database structures and multiple parsing passes.
* **Interview-Ready Answer:** "To handle long documents, I construct a hierarchical index: I embed summaries of chapters or sections alongside granular paragraph chunks. This allows us to search the high-level summaries first, and then retrieve only the specific paragraphs needed, keeping prompt sizes small and preserving semantic structure."

---

### Q17: How do you handle contradictory sources?

* **Definition:** Contradictory source handling refers to the strategy of resolving conflicts when different retrieved documents contain mismatching facts (e.g., one file says a fee is $10, and another says it is $12).
* **Practical Solution:**
  1. **Recency Bias:** Include modification timestamps in the chunk metadata. Instruct the LLM: *"If facts contradict, prioritize the information with the most recent timestamp."*
  2. **Authoritative Tagging:** Score source quality (e.g., official manuals vs. Slack logs). Tag chunks with an authority score and instruct the LLM to prioritize official sources.
  3. **Expose Conflict:** Instruct the model to highlight the contradiction to the user (e.g., *"Source A states X, but Source B states Y"*), rather than guessing which one is correct.
* **Tradeoff:** Highlighting conflicts increases transparency and user trust but can make responses look less decisive.
* **Interview-Ready Answer:** "I handle contradictory sources by tagging chunks with metadata timestamps and authority scores. The prompt template instructs the LLM to prioritize the most recent or official source, and if the contradiction cannot be resolved, to explicitly present both perspectives to the user rather than guessing."

---

### Q18: How do you add citations?

* **Definition:** Citation engineering is the practice of structuring prompts and post-processing tools to ensure the LLM links its assertions to specific retrieved documents.
* **Practical Solution:**
  1. Format retrieved chunks with unique keys (e.g., `<doc id="doc_3">`).
  2. Instruct the LLM: *"You must cite your sources inline using the format [doc_id] for every factual claim. Never make a claim without an inline citation."*
  3. Write a regex parser on the backend to validate that all generated citation IDs exist in the retrieved context set.
  4. Convert the citation brackets into clickable UI links in the frontend.
* **Tradeoff:** Enforcing strict citations reduces hallucinations but can occasionally result in formatting errors or slightly degrade generation flow.
* **Interview-Ready Answer:** "I implement citations by wrapping context chunks in XML tags with unique IDs and instructing the LLM to insert these IDs inline as bracketed markers. We run a backend validation step to confirm the cited IDs exist in the retrieved set before rendering them as clickable links in the UI."

---

### Q19: How do you detect bad retrieval?

* **Definition:** Bad retrieval detection is the runtime logic that flags when the retrieved context does not contain relevant information to answer the user's query, preventing the model from writing hallucinated answers.
* **Practical Solution:**
  1. **Cosine Similarity Threshold:** If the top-1 retrieved chunk has a cosine similarity score below a threshold (e.g., 0.65), flag the retrieval as low-confidence.
  2. **Rerank Score Threshold:** If the top-ranked chunk from the Cross-Encoder has a score below 0.3, flag it.
  3. **LLM Validation:** Instruct the LLM to evaluate the relevance of the context first. If the context is irrelevant, output a fallback response: *"I cannot find the answer in the provided documents."*
* **Tradeoff:** Setting high thresholds prevents hallucinations but can increase false-positive refusals if the embedding model uses a different distance scale.
* **Interview-Ready Answer:** "I detect bad retrieval by setting minimum score thresholds on both our first-stage vector search and our reranker. If the top match falls below these thresholds, or if the LLM flags the context as irrelevant, we abort the generation and output a safe fallback response."

---

### Q20: How do you improve retrieval quality?

* **Definition:** Retrieval quality improvement refers to optimizing the precision and recall of the search phase to ensure the most relevant documents are passed to the generator.
* **Practical Solution:**
  1. **Hybrid Search:** Combine BM25 keyword matching with dense vector search.
  2. **Query Reformulation:** Use a fast LLM to rewrite user queries, resolving typos and conversational references.
  3. **Cross-Encoder Reranking:** Use a reranker model to re-sort the top retrieved chunks based on token-to-token similarity.
  4. **Metadata Filtering:** Restrict search parameters using categories, dates, or user roles.
* **Tradeoff:** Improving retrieval quality increases query pipeline complexity and adds latency, but it directly minimizes hallucinations.
* **Interview-Ready Answer:** "I improve retrieval quality by implementing hybrid search to combine keyword and semantic signals, using query rewriting to resolve user typos and conversational references, and running a Cross-Encoder reranker to position the most relevant documents at the top of the context window."

---

### Q21: What is reranking and when should you use it?

* **Definition:** Reranking is a two-stage retrieval process where a fast retriever fetches a candidate pool of documents, and a computationally heavy Cross-Encoder model re-scores and re-orders those candidates.
* **Practical Solution:** 
  - I deploy a reranker model (like Cohere Rerank or BGE-Reranker) in all production RAG systems where accuracy is critical.
  - The first-stage search retrieves 50-100 candidates.
  - The reranker re-scores the candidates and returns only the top 5 to the LLM.
* **Tradeoff:** Reranking adds 50-150ms of latency to the query loop, but it significantly improves generation accuracy and allows you to feed fewer, higher-quality tokens to the LLM, reducing generation cost.
* **Interview-Ready Answer:** "Reranking uses a Cross-Encoder model to re-evaluate and re-sort an initial pool of candidate documents. I use it in production to place the most relevant facts at the top of the context window, which improves generation accuracy and allows us to feed fewer tokens to the LLM."

---

### Q22: What is hybrid search?

* **Definition:** Hybrid search is the integration of sparse keyword search (BM25) and dense vector search (semantic embeddings) within a single query pass.
* **Practical Solution:** 
  1. Create a sparse index (using BM25 in OpenSearch or pg_trgm in Postgres) and a dense index (in Qdrant or Pinecone).
  2. Query both indexes in parallel.
  3. Merge the results using Reciprocal Rank Fusion (RRF).
* **Tradeoff:** Hybrid search increases database configuration complexity and storage overhead, but it is necessary to catch both exact keyword matches and semantic concepts.
* **Interview-Ready Answer:** "Hybrid search combines sparse keyword search and dense vector search in parallel, merging their results using Reciprocal Rank Fusion. This provides high precision for exact terms—like serial numbers or acronyms—and high recall for semantic concepts."

---

### Q23: What is query expansion?

* **Definition:** Query expansion (or multi-query) is the practice of generating multiple variations of a user's prompt to increase retrieval recall by searching from different perspectives.
* **Practical Solution:**
  1. Prompt a fast LLM: *"Generate 3 different search queries to find documents for: [User Query]"*.
  2. Execute vector search queries for all generated variations in parallel.
  3. Consolidate and deduplicate the retrieved document lists before passing them to the reranker.
* **Tradeoff:** Query expansion maximizes search coverage (high recall) but increases database read loads by 4x and adds minor pre-processing latency.
* **Interview-Ready Answer:** "Query expansion uses a lightweight LLM to generate multiple variations or synonyms of the user's query. We run search queries for all variations in parallel and merge the results, which maximizes our retrieval coverage and prevents missing relevant files."

---

### Q24: What is query rewriting?

* **Definition:** Query rewriting is a preprocessing step that reformulates conversational, pronoun-heavy, or messy user prompts into clean, search-optimized queries.
* **Practical Solution:**
  1. Pass the user's query and the chat history to a fast model.
  2. Prompt the model to resolve pronouns (e.g., converting "How do I fix it?" to "How do I fix the database connection timeout error").
  3. Send only the cleaned, rewritten query to the retriever.
* **Tradeoff:** Query rewriting is necessary for multi-turn chat assistants to maintain context, but adds a preprocessing LLM call, increasing latency by 100-300ms.
* **Interview-Ready Answer:** "Query rewriting uses a fast LLM to translate conversational, pronoun-heavy user prompts into clean, search-optimized queries based on the chat history. This ensures that search queries carry the full context of the ongoing conversation."

---

### Q25: What is metadata filtering?

* **Definition:** Metadata filtering is the application of hard SQL-like constraints (such as filtering by date, category, or user permissions) to the search query, restricting the vector search space.
* **Practical Solution:** 
  1. Tag text chunks with structured fields (e.g., `{"department": "finance", "year": 2024}`) during ingestion.
  2. Construct the search query to include these filters alongside the vector.
  3. The vector database runs a single-stage query to execute the filters and vector search simultaneously.
* **Tradeoff:** Metadata filtering ensures query results conform to business constraints and security permissions, but requires maintaining and indexing structured metadata attributes.
* **Interview-Ready Answer:** "Metadata filtering applies hard logical constraints—like date ranges, document categories, or security permissions—to a search query. Modern vector databases execute these filters and the vector search in a single stage, ensuring results conform to business constraints."
