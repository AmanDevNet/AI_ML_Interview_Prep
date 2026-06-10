# 📓 Phase 6 Chapter 1: Beginner & Intermediate Q&A

This chapter contains 35 screening and technical questions (Questions 1 to 35) covering the core principles of LLMs and RAG pipelines.

---

## 🟢 Beginner Questions (Q1 - Q20)

### Q1: What is an LLM?
* **Short Answer:** A Large Language Model is a deep learning model trained on massive amounts of text to predict the next token in a sequence.
* **Detailed Explanation:** LLMs are based on the Transformer decoder architecture. By learning next-token probabilities over trillions of words, they compress human grammar, code logic, and general facts into billions of weight matrices, enabling zero-shot reasoning.
* **Example:** GPT-4 or Llama 3 predicting the word "learning" in the sentence "Deep learning models are..."
* **Follow-up Question:** How does the scaling of parameter size affect model reasoning capabilities?

---

### Q2: What is a token?
* **Short Answer:** A token is a text unit (a character, syllable, or word fragment) that is mapped to a unique integer ID for the model to process.
* **Detailed Explanation:** LLMs do not read letters. Tokenizers (like BPE) break text into sub-words. On average, 1 token equals 0.75 English words or 4 characters.
* **Example:** The word "unhelpful" might be split into two tokens: `["un", "helpful"]` and mapped to IDs like `[1294, 3810]`.
* **Follow-up Question:** Why do token counts vary widely across different languages?

---

### Q3: How does a model read words?
* **Short Answer:** A model converts word tokens into dense embedding vectors representing their positions in a high-dimensional semantic map.
* **Detailed Explanation:** Integer token IDs index a lookup matrix to extract static embeddings. Positional encodings are added to track word order, and attention layers dynamically update these vectors based on context.
* **Example:** Resolving "bank" to a financial institution when "money" appears nearby in the text.
* **Follow-up Question:** What positional encoding method is used in modern Llama models?

---

### Q4: What is the vocabulary of an LLM?
* **Short Answer:** The vocabulary is the fixed set of unique tokens the model's tokenizer is trained to recognize.
* **Detailed Explanation:** Vocab size is set before pre-training (typically 32k to 256k tokens). It bounds the size of the model's input embedding and final projection matrices.
* **Example:** If "ChatGPT" isn't in the vocabulary, it is split into sub-word tokens: `["Chat", "G", "PT"]`.
* **Follow-up Question:** What are the computational downsides of a very large vocabulary?

---

### Q5: What is an embedding?
* **Short Answer:** An embedding is a dense vector representation of text where semantic meaning corresponds to spatial distance in a continuous vector space.
* **Detailed Explanation:** Text is mapped to a fixed-length float array (e.g., 1536 dimensions). Similarity is computed via mathematical dot products or angles, bypassing string matching.
* **Example:** Vectors for "cat" and "kitten" sit close together, while "computer" is far away.
* **Follow-up Question:** What is the difference between sparse and dense embeddings?

---

### Q6: What is a context window?
* **Short Answer:** The context window is the maximum sequence length (input prompt + generated output) the model can process at once.
* **Detailed Explanation:** Context is limited by the $O(N^2)$ scaling of attention compute and GPU memory (VRAM). Exceeding it triggers out-of-memory errors or truncation.
* **Example:** Passing a 50-page PDF (approx. 40,000 tokens) into a model with an 8,000 token limit will fail.
* **Follow-up Question:** How does KV cache scale inside a long-context window?

---

### Q7: What is prompt engineering?
* **Short Answer:** Prompt engineering is the optimization of input layouts, instructions, and examples to guide model outputs without changing weights.
* **Detailed Explanation:** It uses structured designs (role prompting, few-shot examples, delimiters) to align query semantics with pre-training associations.
* **Example:** Adding "Think step-by-step before answering" to force the model to output a logical chain of thought.
* **Follow-up Question:** What is the difference between prompting and in-context learning?

---

### Q8: What is the system prompt?
* **Short Answer:** The system prompt is a high-priority instruction set placed at the start of the context to define model persona, rules, and safety boundaries.
* **Detailed Explanation:** Aligned models use special tokens to separate system instructions from user inputs, instructing the attention layers to prioritize system guidelines.
* **Example:** *"You are a banking assistant. You speak only in JSON. Never output code."*
* **Follow-up Question:** How does prompt injection bypass system prompt constraints?

---

### Q9: What is temperature?
* **Short Answer:** Temperature is a logit-scaling parameter that controls the randomness and diversity of token generation.
* **Detailed Explanation:** Dividing logits by temperature before Softmax alters the distribution. Lower values ($<0.2$) focus probability on the top choice; higher values ($>0.7$) flatten it.
* **Example:** Setting $T=0$ for code syntax validation, and $T=0.8$ for copywriting.
* **Follow-up Question:** Why is setting temperature to exactly 0 mathematically undefined?

---

### Q10: What is Top-p?
* **Short Answer:** Top-p (nucleus sampling) selects from a dynamic pool of tokens whose cumulative probability meets the threshold $p$.
* **Detailed Explanation:** Tokens are sorted by probability. We sum them until reaching $p$ (e.g., 0.9), discard all other options, and renormalize the remaining pool.
* **Example:** If $p=0.9$ and the top token has a probability of 0.95, the pool is restricted to just that 1 token.
* **Follow-up Question:** How does Top-p differ from Top-k sampling?

---

### Q11: What is the maximum tokens parameter?
* **Short Answer:** Max tokens is an inference constraint that stops the generation loop once the output reaches the specified token count limit.
* **Detailed Explanation:** The engine counts generated tokens and terminates when the count is hit or when the EOS (end-of-sequence) token is generated.
* **Example:** Limiting summaries to exactly 100 tokens to fit a UI card layout.
* **Follow-up Question:** What does a `finish_reason` of `"length"` indicate in an API response?

---

### Q12: What is a hallucination?
* **Short Answer:** A hallucination is when an LLM generates factually incorrect, self-contradictory, or ungrounded text that sounds fluent.
* **Detailed Explanation:** LLMs maximize token likelihood, not factual correctness. They cannot verify truth and will guess plausible continuations when facing low-confidence states.
* **Example:** A model creating a fake biography or referencing a non-existent scientific paper.
* **Follow-up Question:** How does RAG help reduce model hallucinations?

---

### Q13: What is RAG?
* **Short Answer:** Retrieval-Augmented Generation is an architecture that queries an external database for facts, inserts them into the prompt, and directs the LLM to write the response.
* **Detailed Explanation:** It splits the system into retrieval (finding documents) and generation (synthesizing answers), turning the LLM into an open-book synthesizer.
* **Example:** A medical bot retrieving a patient's chart and the latest clinical trial PDF to write a treatment draft.
* **Follow-up Question:** What is the difference between pre-retrieval and post-retrieval optimization?

---

### Q14: Why do we use RAG instead of just prompting?
* **Short Answer:** Prompting is bounded by context window limits; RAG allows querying search indexes across petabytes of files.
* **Detailed Explanation:** Prompting requires hard-coding data. RAG dynamically filters down huge datasets to pass only the top-k relevant chunks to the prompt.
* **Example:** Querying a search index of 100,000 corporate policy files, rather than trying to paste all files into the prompt.
* **Follow-up Question:** How does RAG impact runtime latency compared to simple prompting?

---

### Q15: What is a vector database?
* **Short Answer:** A vector database is a data store optimized for storing, indexing, and executing fast similarity queries over high-dimensional vector arrays.
* **Detailed Explanation:** Standard DBs index scalars. Vector DBs use Approximate Nearest Neighbor (ANN) graphs (like HNSW) to search millions of vectors in milliseconds.
* **Example:** Using Pinecone, Qdrant, or PGVector to index and search document embeddings.
* **Follow-up Question:** What is the purpose of scalar quantization in vector databases?

---

### Q16: What is a chunk?
* **Short Answer:** A chunk is a single, self-contained segment of text split from a larger document prior to vectorization.
* **Detailed Explanation:** Splitting text ensures semantic focus. Chunks are the base nodes stored in the database and retrieved at search time.
* **Example:** Slicing a 10-page user manual into 50 paragraphs of 300 words each.
* **Follow-up Question:** How do you preserve structural headers during chunking?

---

### Q17: What is chunk size?
* **Short Answer:** Chunk size is the hyperparameter specifying the maximum token or character count allowed in a single chunk.
* **Detailed Explanation:** It determines the semantic scope of the vector. Correct tuning balances search precision with the context needed to answer.
* **Example:** Setting a chunk limit of 512 tokens to match the maximum input length of a sentence transformer.
* **Follow-up Question:** What is the risk of using very large chunk sizes?

---

### Q18: What is chunk overlap?
* **Short Answer:** Chunk overlap is the duplicate boundary region shared between adjacent chunks to prevent cutting semantic thoughts in half.
* **Detailed Explanation:** It copies tokens from the end of one chunk to the start of the next, maintaining continuity across boundaries.
* **Example:** Slicing text with chunk size 500 and overlap 50: Chunk 1 spans 0-500, Chunk 2 spans 450-950.
* **Follow-up Question:** What is the tradeoff of having a very large chunk overlap?

---

### Q19: What is similarity search?
* **Short Answer:** Similarity search is the mathematical calculation used to identify vectors in a database that sit closest to a query vector.
* **Detailed Explanation:** It computes distances using cosine similarity, dot product, or L2 distance, ranking documents based on semantic closeness.
* **Example:** Matching a vector for "How do I reset my password?" to a chunk vector for "Credential recovery steps."
* **Follow-up Question:** Which distance metric is best for unnormalized embeddings?

---

### Q20: What is top-k in retrieval?
* **Short Answer:** Top-k is the parameter specifying the exact number of highest-scoring chunks retrieved from the database to be sent to the prompt.
* **Detailed Explanation:** It bounds the context volume. It balances retrieval recall with LLM token costs and latency constraints.
* **Example:** Setting $k=5$ to retrieve only the 5 most relevant pages from a company wiki.
* **Follow-up Question:** How does the choice of $k$ affect generator latency?

---

## 🟡 Intermediate Questions (Q21 - Q35)

### Q21: What is the difference between RAG and fine-tuning?
* **Short Answer:** RAG updates context at query time without updating weights; fine-tuning modifies model parameters to learn specific styles or syntax.
* **Detailed Explanation:** RAG is a non-parametric update suitable for factual accuracy, dynamic updates, and data security. Fine-tuning is a parametric update that alters model behavior and styling.
* **Example:** Fine-tuning a model to output SQL code, and wrapping it in RAG to retrieve table schemas at runtime.
* **Follow-up Question:** How do you handle a scenario where a database changes its contents hourly?

---

### Q22: How does BPE tokenization handle misspelled or rare words?
* **Short Answer:** BPE handles unknown words by breaking them down into constituent sub-word prefixes, roots, or individual characters that exist in its vocabulary.
* **Detailed Explanation:** The vocabulary contains root characters. If a word is rare, the BPE tokenizer splits it recursively until finding matching pieces, avoiding Out-of-Vocabulary errors.
* **Example:** Misspelling "elephant" as "elephhant" might split it into: `["ele", "ph", "h", "ant"]`.
* **Follow-up Question:** How does byte-level fallback improve tokenization safety?

---

### Q23: Explain the difference between Cosine similarity and Dot Product similarity.
* **Short Answer:** Cosine similarity measures the angle between vectors (length independent); Dot Product measures both angle and magnitude.
* **Detailed Explanation:** Cosine similarity divides the dot product by vector magnitudes. If vectors are pre-normalized to length 1, they are mathematically identical.
* **Example:** When using normalized embeddings, the dot product is faster because it bypasses division operations on the CPU/GPU.
* **Follow-up Question:** Why are unnormalized vectors a problem for dot product similarity?

---

### Q24: What is the difference between an Encoder-only and Decoder-only model?
* **Short Answer:** Encoder-only models allow bidirectional attention (great for understanding); Decoder-only models use causal masking (great for generation).
* **Detailed Explanation:** Encoders (like BERT) process all tokens simultaneously, mapping contextual relationships. Decoders (like GPT) mask future tokens to predict outputs sequentially.
* **Example:** Using BERT for embedding search indexing, and GPT-4 for text synthesis generation.
* **Follow-up Question:** Why did decoder-only models win the scaling race for general assistants?

---

### Q25: What is a recursive character splitter?
* **Short Answer:** A text splitter that splits paragraphs and sentences using a hierarchy of separators to keep semantic thoughts intact.
* **Detailed Explanation:** It uses separators like `["\n\n", "\n", " ", ""]` sequentially. It only splits on smaller separators if the text block exceeds the target chunk size.
* **Example:** Keeping a complete paragraph in one chunk instead of slicing it mid-sentence.
* **Follow-up Question:** What is the fallback separator if a word itself exceeds the chunk size?

---

### Q26: What is semantic chunking?
* **Short Answer:** An advanced chunking strategy that splits documents based on topic transitions rather than token limits.
* **Detailed Explanation:** It splits text into sentences, embeds them, calculates cosine distance between adjacent sentences, and splits where semantic distance exceeds a threshold.
* **Example:** Slicing a medical guideline strictly at the boundary between "Dosage" and "Side Effects."
* **Follow-up Question:** How does semantic chunking affect database ingestion latency?

---

### Q27: What is hybrid search?
* **Short Answer:** The retrieval technique that queries both a keyword index (BM25) and a semantic vector index, merging the outputs.
* **Detailed Explanation:** It combines exact keyword match precision with embedding semantic match recall, using Reciprocal Rank Fusion (RRF) to merge lists.
* **Example:** Querying "CVE-2024-3011" and catching both the exact vulnerability ID and semantic security articles.
* **Follow-up Question:** How do you configure RRF parameter $k$ to balance sparse and dense weight priorities?

---

### Q28: What is BM25 keyword search?
* **Short Answer:** A keyword matching algorithm that ranks documents based on term frequency, inverse document frequency, and document length normalization.
* **Detailed Explanation:** It improves upon TF-IDF by adding term frequency saturation: after a threshold, repeating a search term yields diminishing score gains.
* **Example:** Preventing a document that repeats the query keyword 100 times from scoring higher than a short, exact matching essay.
* **Follow-up Question:** What is the significance of the $k_1$ parameter in the BM25 formula?

---

### Q29: What is Reranking?
* **Short Answer:** A second-stage search phase where a Cross-Encoder evaluates the exact alignment of a query and retrieved documents, re-sorting them.
* **Detailed Explanation:** Rerankers compute token-to-token attention across query and document together. They are slow but precise, so they are applied only to the top candidates.
* **Example:** Fetching 100 candidate chunks via vector search, and using Cohere Rerank to select the top 5 to pass to the LLM.
* **Follow-up Question:** How does reranking affect overall system latency?

---

### Q30: What is the "Lost in the Middle" phenomenon?
* **Short Answer:** The tendency of LLMs to ignore or fail to retrieve information placed in the middle of long input prompts.
* **Detailed Explanation:** Attention layers focus heavily on the extreme beginning (praying bias) and end (recency bias) of context strings.
* **Example:** A model failing to answer a question when the target fact is located on page 14 of a 30-page prompt context.
* **Follow-up Question:** How do you structure context assembly to mitigate this issue?

---

### Q31: What is query rewriting?
* **Short Answer:** Using a fast model to translate messy or conversational prompts into optimized search strings.
* **Detailed Explanation:** It resolves pronouns and extracts core search intents by analyzing historical chat sequences before querying the database.
* **Example:** Rewriting "What about its pricing?" in a conversation about Slack to "Slack corporate pricing plans."
* **Follow-up Question:** What is the risk of query rewriting on latency?

---

### Q32: Explain Multi-query retrieval.
* **Short Answer:** Query expansion where an LLM generates multiple distinct search variations of a query to maximize retrieval recall.
* **Detailed Explanation:** It queries the database for multiple synonyms, aggregates the returned document lists, and deduplicates the results.
* **Example:** Expanding "database scaling" to "database sharding," "horizontal scaling," and "partitioning strategies."
* **Follow-up Question:** How does multi-query affect database read throughput limits?

---

### Q33: What is the RAG Triad of evaluation?
* **Short Answer:** An evaluation framework that measures Context Relevance, Groundedness (Faithfulness), and Answer Relevance.
* **Detailed Explanation:**
  - Context Relevance checks the retriever.
  - Groundedness checks if the generator hallucinated.
  - Answer Relevance checks if the LLM answered the query.
* **Example:** Using Ragas to identify that a chatbot is hallucinating because its Groundedness score dropped to 0.4.
* **Follow-up Question:** If Groundedness is low but Context Relevance is high, what is failing?

---

### Q34: How does prompt caching reduce latency and cost?
* **Short Answer:** It stores precomputed KV states of static prompt prefixes in GPU memory, skipping recalculations on repeated calls.
* **Detailed Explanation:** The prefill phase of the prompt is cached. Subsequent queries sharing the same prefix (like system instructions) bypass prefill compute.
* **Example:** Caching a 5,000-token system instruction booklet on Anthropic APIs, saving 90% of prefill latency.
* **Follow-up Question:** What is the minimum prompt prefix token length required to trigger caching on Anthropic APIs?

---

### Q35: What is structured output and why is it needed in production?
* **Short Answer:** Forcing an LLM to generate replies conforming to a strict schema (JSON/XML) for down-stream database parsing.
* **Detailed Explanation:** Free-form text changes format. Structured outputs modify sampling logits at runtime (using context-free grammar FSMs) to guarantee parsing validity.
* **Example:** Forcing a product extraction bot to output exactly `{"product_name": str, "price": float}`.
* **Follow-up Question:** What is the difference between regex-constrained sampling and LLM-based parsing retries?
