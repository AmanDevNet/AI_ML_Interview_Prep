# 📗 Phase 2 Chapter 2: Ingestion & Preprocessing

This chapter covers the critical first mile of RAG: how raw enterprise documents are loaded, cleaned, parsed, split into logical segments, and enriched with metadata before indexing.

---

## 📋 Topics Covered
6. [Topic 6: Document Ingestion](#topic-6-document-ingestion)
7. [Topic 7: Document Parsing](#topic-7-document-parsing)
8. [Topic 8: Text Cleaning](#topic-8-text-cleaning)
9. [Topic 9: Chunking](#topic-9-chunking)
10. [Topic 10: Chunk Size](#topic-10-chunk-size)
11. [Topic 11: Chunk Overlap](#topic-11-chunk-overlap)
12. [Topic 12: Semantic Chunking](#topic-12-semantic-chunking)
13. [Topic 13: Metadata](#topic-13-metadata)

---

### Topic 6: Document Ingestion

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Document Ingestion is the pipeline component responsible for monitoring, extracting, and loading raw source files from data repositories (e.g., S3, Google Drive, databases, Slack channels) into the RAG system.
* **Why it matters:** Ingestion handles connectivity, sync timing, and change tracking. Without an ingestion pipeline, you cannot update your RAG database when source documents are added, edited, or deleted.
* **How it affects answer quality:** It affects the freshness and completeness of answers. If the ingestion pipeline is slow or skips files, the LLM will generate answers based on outdated or missing information.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Push vs. Pull updates. A push-based system uses webhooks to trigger ingestion immediately when a file changes (low latency, high api configuration complexity). A pull-based system polls the repository at intervals (simpler, but introduces sync delays).
* **Production Example:** A Slack bot RAG application. The ingestion service runs on a webhook that fires whenever a new message is posted in designated company channels, instantly routing the text to the RAG preprocessing pipeline.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Ingestion is the data connector layer that pulls raw files from company storage systems into the RAG environment, handling change detection and indexing schedules."
* **Top Q&As:**
  * **Question 1:** How do you handle document deletion in an ingestion pipeline?
    * **Answer:** We must implement a deletion sync mechanism. When a file is marked deleted in the source directory, the ingestion layer tracks its unique document ID, queries the vector database for all vectors associated with that document ID, and issues a hard delete command to clear those vectors.
    * **Follow-up:** How do you avoid re-processing unmodified files? (Answer: We store cryptographic hashes—like MD5—of each file's contents. During the next ingestion run, we compare hashes; if they match, we skip parsing and embedding for that file).

---

### Topic 7: Document Parsing

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Document Parsing is the extraction of raw text, structural tags, and tables from binary file formats (like PDF, DOCX, HTML, PPTX, or scanned images).
* **Why it matters:** Real-world documents are highly structured. Simply scraping raw characters strips away structural hierarchy, column formats, tables, and headers, turning the data into confusing strings.
* **How it affects answer quality:** High. If a parser fails to extract tables cleanly or parses multi-column PDFs horizontally instead of vertically, the resulting text chunks will contain mixed sentences, causing the LLM to generate inaccurate answers.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Rule-based Parsing vs. Layout-Aware ML Models. Rule-based libraries (like PyPDF) are extremely fast and free, but struggle with tables and columns. Vision-based models (like LayoutLM or Unstructured) are highly accurate on complex pages but are slow and require GPU compute.
* **Production Example:** An insurance company parsing claims documents. The pipeline uses a vision-based layout parser (like Marker or Nougat) to extract tables, footnotes, and diagrams as clean Markdown formats, preserving semantic structure.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Parsing extracts clean text and structural markers from complex files like PDFs. To preserve meaning in production, we use layout-aware parsers that convert columns, headers, and tables into markdown."
* **Top Q&As:**
  * **Question 1:** Why is parsing tables in PDFs so difficult for basic RAG systems, and how do you solve it?
    * **Answer:** Basic PDF parsers extract text by tracking coordinate lines, which mixes columns and rows into a single unreadable string. We solve this by using layout-aware vision models to detect tables, extract them into structured markdown tables (e.g. `| Col 1 | Col 2 |`), or use OCR systems to output HTML tables. The model can then easily read the structured rows.
    * **Follow-up:** What if the table is too complex for Markdown? (Answer: We crop the table as an image, use a vision-to-text model like GPT-4o to write a detailed textual summary of the table, embed that summary, and link it to the table image).

---

### Topic 8: Text Cleaning

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Text Cleaning is the normalization of raw extracted text, removing boilerplate content (headers, footers, page numbers, formatting noise) and correcting character encoding issues (like broken ligatures or bad Unicode mappings).
* **Why it matters:** Parsed text is often cluttered with repetitive boilerplate (e.g., "Page 3 of 50 - Confidential"). Leaving this in the text adds noise to embeddings, inflates token counts, and wastes context space.
* **How it affects answer quality:** Cleaning improves retrieval accuracy. Removing boilerplate prevents the retriever from selecting chunks based on matching headers/footers instead of the actual content.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Heavy Regex/Cleaning vs. Content Loss. Aggressive cleaning regexes can accidentally delete valid abbreviations, equations, or code strings, ruining technical documentation.
* **Production Example:** Cleaning web scraped pages by removing HTML tags, JavaScript blocks, navigation bars, and cookie consent banners, keeping only the main article text body.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Text cleaning strips out noise like page numbers, repetitive headers, HTML tags, and formatting glitches, ensuring the vector database only indexes clean, semantic content."
* **Top Q&As:**
  * **Question 1:** How do you programmatically remove headers and footers from parsed PDF pages without deleting matching text in the body?
    * **Answer:** During the parsing step, we extract the structural coordinates of the text. Because headers and footers appear at fixed coordinates at the extreme top and bottom of every page, we write coordinate-based filters to drop text appearing in those margins before merging the document content.
    * **Follow-up:** What if the document has variable margins? (Answer: We run a sequence similarity pass over the first and last lines of all pages; lines that repeat across multiple pages with high similarity are flagged as boilerplate and removed).

---

### Topic 9: Chunking

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Chunking is the process of splitting a long document into smaller, self-contained textual segments (chunks) prior to embedding and indexing.
* **Why it exists:** LLMs have limited context windows and retrieval search is most effective when searching for narrow, focused ideas. Sending an entire 100-page document as a single search result would exceed token limits and dilute semantic representations.
* **How it works:**
  1. Text is processed sequentially.
  2. Boundaries are determined by characters, words, sentences, or paragraphs.
  3. Chunking rules split the text (e.g., every 500 characters or whenever a paragraph break `\n\n` is detected).
  4. Each chunk is saved as an individual node in the database.
* **Simple Analogy:** Chunking is like cutting a long scroll of text into individual, readable flashcards. Each card should contain a single, complete thought.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Splitting a software manual into chunks based on markdown header boundaries (`#`, `##`, `###`), ensuring each API function is stored in its own dedicated chunk.
* **Engineering Tradeoffs:** Small Chunks vs. Large Chunks. Small chunks (e.g., 100 tokens) yield very precise search results but lack context, leading to poor answers. Large chunks (e.g., 2000 tokens) preserve rich context but dilute embedding vectors, making them harder to find via similarity search.
* **System Design Relevance:** Chunk size determines the index size of your vector database and directly affects the input token cost of your LLM calls.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Chunking is the division of long documents into smaller, semantic segments. It ensures that the vector search index can match granular queries precisely without overloading the generator's context window."
* **Common Candidate Mistakes:** Assuming character-based splitting is sufficient (character splits will slice words and numbers in half, breaking semantic meaning; token-based or sentence-based chunking is mandatory).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is recursive character chunking?
    * **Answer:** It is a chunking method that uses a list of separators (typically `["\n\n", "\n", " ", ""]`) in order of priority. It attempts to split text using the largest separator first (paragraphs). If a paragraph is still larger than the target chunk size, it splits it using the next separator (sentences), and then words, preserving the structure as much as possible.
    * **Follow-up:** Why is this preferred over fixed-character chunking? (Answer: Because it avoids splitting paragraphs or sentences mid-thought, keeping semantic units intact).

---

### Topic 10: Chunk Size

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Chunk Size is the parameter specifying the maximum number of tokens or characters allowed in a single chunk.
* **Why it matters:** Choosing the right chunk size is a key tuning hyperparameter. It directly balances search precision (finding the needle) against generation coherence (having enough context to answer).
* **How it affects answer quality:** If chunk sizes are too small (e.g., 50 tokens), critical surrounding details are lost, and the LLM cannot synthesize a complete answer. If they are too large (e.g., 1500 tokens), irrelevant text is included, diluting the search signal and cluttering the context window.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Configuring a RAG system for legal contracts requires larger chunk sizes (e.g., 800 tokens) because clauses contain highly nested, cross-referenced definitions. Configuring it for a customer Q&A database can use smaller chunks (e.g., 200 tokens).
* **Engineering Tradeoffs:** Compute/Storage vs. Performance. Smaller chunks generate more total vector nodes, raising index storage costs and search compute times. Larger chunks save index space but consume more input tokens during LLM generation.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Chunk size dictates the token capacity of each indexed document segment. It is a critical hyperparameter that balances granular search precision against the semantic context required by the LLM to generate answers."
* **Common Candidate Mistakes:** Choosing an arbitrary chunk size (like 512 tokens) without analyzing the average length of semantic thoughts in the specific document domain.
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** How do you empirically determine the optimal chunk size for a new RAG application?
    * **Answer:** We run an evaluation loop. We create a test dataset of representative queries and answers. We run our RAG pipeline using different chunk sizes (e.g., 128, 256, 512, 1024) and measure **context recall** (did we retrieve the answer?) and **context precision** (how much noise did we retrieve?). We select the size that optimizes both metrics.
    * **Follow-up:** How does your embedding model's limit affect chunk size? (Answer: The chunk size must be smaller than the maximum input limit of the embedding model, which is typically 512 or 8192 tokens; exceeding this limit will truncate the text, leading to unindexed data).

---

### Topic 11: Chunk Overlap

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Chunk Overlap is the parameter that specifies how many tokens are shared between adjacent chunks (e.g., a chunk size of 500 with an overlap of 50 tokens).
* **Why it exists:** When text is split, critical context might get cut in half at the boundary. For example, if a key sentence starts at the end of Chunk 1 and finishes in Chunk 2, the semantic meaning of that sentence is lost in both.
* **How it works:**
  1. Chunk 1 is created from token 0 to 500.
  2. With a 50-token overlap, Chunk 2 is created from token 450 to 950.
  3. Token 450 to 500 exists in both chunks, preserving semantic links across the split boundary.
* **Simple Analogy:** Imagine cutting a strip of pictures. If you cut exactly at the edges, you might slice someone's face in half. By overlapping the cuts slightly, you make sure the complete subject appears in at least one frame.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Processing narrative text or history books, where events flow continuously. An overlap of 10-20% ensures that names, dates, and transitional pronouns remain linked to their actions in adjacent chunks.
* **Engineering Tradeoffs:** Redundancy vs. Context. Higher overlap (e.g., 40%) preserves maximum contextual continuity but increases index storage overhead and can lead to the LLM receiving duplicate text blocks in the retrieved prompt.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Chunk overlap duplicates tokens across adjacent boundaries. It prevents semantic loss by ensuring that sentences and concepts split at the chunk edge are fully captured in at least one adjacent chunk."
* **Common Candidate Mistakes:** Setting overlap to 0, which commonly leads to retrieval failures for queries targeting boundary-spanning facts.
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the typical rule of thumb for setting chunk overlap?
    * **Answer:** The industry standard is typically between 10% and 20% of the chunk size. For instance, a chunk size of 500 tokens is commonly paired with a 50-to-100 token overlap.
    * **Follow-up:** If we use paragraph-based chunking, do we still need overlap? (Answer: We need less overlap because paragraphs are natural semantic boundaries, but a small sentence overlap is still helpful to maintain narrative flow).

---

### Topic 12: Semantic Chunking

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Semantic Chunking is an advanced splitting strategy that groups text into chunks dynamically based on changes in semantic meaning, rather than relying on fixed character or token counts.
* **Why it exists:** Fixed-size chunking is arbitrary and often splits single ideas apart. Semantic chunking ensures that each chunk represents a single, complete topic, improving the clarity of the vector representation.
* **How it works:**
  1. Split the document into individual sentences.
  2. Generate embeddings for each sentence using a fast embedding model.
  3. Calculate the cosine distance between the embeddings of consecutive sentences.
  4. If the distance exceeds a calculated threshold (representing a change in topic), split the document and start a new chunk.
* **Simple Analogy:** Imagine reading a transcript and grouping paragraphs together. Whenever the speaker shifts from talking about "pricing" to "system architecture," you draw a line to start a new chapter.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Processing medical research papers. Instead of splitting every 500 words, semantic chunking cleanly separates the "Methodology," "Results," and "Conclusion" sections because the vectors of these sections are distinct.
* **Engineering Tradeoffs:** Accuracy vs. Performance. Semantic chunking aligns chunks perfectly with ideas, improving retrieval accuracy. However, generating embeddings for every individual sentence to calculate split points makes ingestion significantly slower and more expensive.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Semantic chunking dynamically splits text based on changes in vector similarity between consecutive sentences. This ensures that chunks are bound by semantic transitions rather than arbitrary token counts, keeping complete thoughts intact."
* **Common Candidate Mistakes:** Assuming semantic chunking can be done without an embedding model (it requires calculating vector distances between sentences).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** How do you set the split threshold in semantic chunking?
    * **Answer:** We typically calculate a relative threshold based on the distribution of differences in the document. For example, we find the 95th percentile of cosine distances between all consecutive sentences in the document and use that as the split threshold.
    * **Follow-up:** How does semantic chunking affect the size of chunks? (Answer: It creates variable-sized chunks. To prevent massive chunks that exceed context limits, we must configure a fallback hard limit to split the chunk if no semantic transition is found within a certain length).

---

### Topic 13: Metadata

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Metadata refers to structured key-value pairs (e.g., author, date, category, URL, access permissions) attached to text chunks during ingestion, stored alongside the vector representation.
* **Why it exists:** Vector search is purely semantic and cannot handle logical rules (like filtering by date, user access, or category). Metadata enables structured filtering to narrow down the search space before running vector math.
* **How it works:**
  1. During ingestion, metadata is extracted from the document properties or path.
  2. When storing the text and vector in the DB, the metadata payload is attached.
  3. At query time, the user sends a filter query: `{"author": "HR", "date": { "$gt": "2023-01-01" }}`.
  4. The database filters out non-matching records and runs the vector search only on the remaining subset.
* **Simple Analogy:** Think of metadata as the labels on file folders. Instead of searching the entire building for a file about "rent," you first filter by "Finance Dept" (metadata filter) and then look through only those folders.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** An enterprise intranet search. Metadata keys like `allowed_groups` (e.g., `['engineering', 'admin']`) are used to prevent unauthorized users from retrieving restricted documents in their search results.
* **Engineering Tradeoffs:** Rich Metadata vs. Storage Costs. Storing large payloads of metadata (like parent document summaries) increases vector database memory usage and index size, raising hosting costs.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Metadata attaches structured attributes to text chunks. It allows us to apply SQL-like filtering constraints (by date, source, or user permissions) before or during vector search to restrict and focus the search space."
* **Common Candidate Mistakes:** Relying on the LLM to filter out outdated or unauthorized documents after retrieval, rather than using metadata filters at the database level (which is a severe security risk and wastes context tokens).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the difference between pre-filtering, post-filtering, and single-stage filtering in vector databases?
    * **Answer:** Pre-filtering applies metadata filters first, then runs vector search on the matches (guarantees top-k matches, but can be slow). Post-filtering runs vector search first, then discards non-matching metadata (can result in fewer than k results). Single-stage filtering (used by modern DBs like Pinecone/Milvus) builds a unified index mapping vectors and metadata, executing both constraints simultaneously for optimal speed and accuracy.
    * **Follow-up:** How do you extract metadata automatically from raw text? (Answer: We use regular expressions for simple patterns like dates or document paths, or run the text through a cheap LLM to extract key entities and save them as metadata keys).
