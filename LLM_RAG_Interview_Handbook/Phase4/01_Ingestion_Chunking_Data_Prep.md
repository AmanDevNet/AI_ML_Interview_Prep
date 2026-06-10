# 📓 Phase 4 Chapter 1: Ingestion, Chunking & Data Prep

This chapter contains deep, production-level answers to critical engineering challenges in document parsing, segmentation, indexing updates, and access security.

---

## 📋 Questions Covered
1. [Q1: How do you choose chunk size?](#q1-how-do-you-choose-chunk-size)
2. [Q2: What happens if chunks are too small?](#q2-what-happens-if-chunks-are-too-small)
3. [Q3: What happens if chunks are too large?](#q3-what-happens-if-chunks-are-too-large)
4. [Q4: How do you handle tables in documents?](#q4-how-do-you-handle-tables-in-documents)
5. [Q5: How do you handle PDFs with poor formatting?](#q5-how-do-you-handle-pdfs-with-poor-formatting)
6. [Q6: How do you handle images/scanned documents?](#q6-how-do-you-handle-imagesscanned-documents)
7. [Q7: How do you handle duplicate documents?](#q7-how-do-you-handle-duplicate-documents)
8. [Q8: How do you update embeddings when documents change?](#q8-how-do-you-update-embeddings-when-documents-change)
9. [Q9: How do you delete documents from a vector DB?](#q9-how-do-you-delete-documents-from-a-vector-db)
10. [Q10: How do you implement access control?](#q10-how-do-you-implement-access-control)

---

### Q1: How do you choose chunk size?

* **Definition:** Chunk size is the maximum token count of text segments. Selecting the size is a database-tuning process that balances search resolution against the context length needed by the generator.
* **Practical Solution:** In production, I determine chunk size using a quantitative sweep:
  1. I extract a representative test set of 100-200 user queries with verified target answers.
  2. I test split configurations: 128, 256, 512, and 1024 tokens (with a fixed 15% overlap).
  3. I use **Ragas** to measure **Context Recall** (did we retrieve the target answer?) and **Context Precision** (how much junk text was retrieved?).
  4. I select the smallest chunk size that maintains a Context Recall of $>90\%$.
* **Tradeoff:** Small chunks (128 tokens) maximize search precision and minimize prompt token bloat, but lose contextual coherence. Large chunks (1024 tokens) preserve narrative flow but dilute vector semantic representations and increase LLM token costs.
* **Interview-Ready Answer:** "I determine chunk size by running an evaluation sweep on a representative golden dataset of queries. I plot context recall and precision across different token sizes—typically 128, 256, 512, and 1024 tokens—and select the smallest size that maintains over 90% context recall to optimize for retrieval precision and keep costs low."

---

### Q2: What happens if chunks are too small?

* **Definition:** Under-chunking occurs when segments are smaller than the natural length of the semantic concepts in the text, resulting in fragments that lack context.
* **Practical Solution:** If chunks are too small (e.g., < 100 tokens), I implement **Parent-Child Retrieval (Sentence Window Retrieval)**:
  1. Store the text as small sentences (child chunks) to optimize vector search precision.
  2. Map each child chunk vector to a larger, surrounding paragraph (parent chunk) in the database metadata.
  3. At search time, query the child chunks but retrieve and feed the larger parent chunk to the LLM.
* **Tradeoff:** Parent-Child retrieval significantly improves answer quality by decoupling search index granularity from generation context, but it increases database index size and storage costs by 2x.
* **Interview-Ready Answer:** "If chunks are too small, the retrieved context loses its semantic integrity, causing the LLM to write incomplete answers or refuse to answer due to missing details. To solve this, I decouple the retrieval representation from the generation representation using parent-child indexing, where we search small sentence vectors but return the larger surrounding paragraph context to the model."

---

### Q3: What happens if chunks are too large?

* **Definition:** Over-chunking occurs when text segments contain multiple distinct topics, diluting the average semantic representation of the vector.
* **Practical Solution:** To mitigate the issues of large chunks (e.g., > 1000 tokens):
  1. Apply **Semantic Chunking** using sentence embedding distance thresholds to ensure chunks only split when a new topic starts.
  2. Use a **Cross-Encoder Reranker** (like Cohere Rerank) to identify the specific sentences inside the large chunk that match the query, stripping out the irrelevant text before building the final prompt.
* **Tradeoff:** Smaller, topic-aligned chunks optimize retrieval, but require more complex parsing pipelines and increase embedding API costs during ingestion.
* **Interview-Ready Answer:** "When chunks are too large, distinct semantic concepts are blended into a single vector, causing query matching to fail because the search signal is diluted by surrounding text. Furthermore, it clutters the LLM's context window with noise and exposes the system to the 'Lost in the Middle' phenomenon."

---

### Q4: How do you handle tables in documents?

* **Definition:** Table parsing is the extraction of grid-based structured data from documents and representing it in a format that LLMs can read.
* **Why it matters:** Text splitters slice tables line-by-line, which destroys the vertical column alignment and mixes cell data into an unreadable string.
* **Practical Solution:** 
  1. Use a layout-aware document parser (like **Unstructured** or **Marker**) to identify table zones.
  2. Extract the tables and convert them to clean Markdown (`| Col 1 | Col 2 |`) or HTML table strings.
  3. Generate a textual summary of each table using a vision-to-text model (like GPT-4o).
  4. Embed the text summary as a chunk, but attach the raw Markdown table in the metadata to be passed to the LLM if the summary is retrieved.
* **Tradeoff:** Summary-based table embedding maximizes search accuracy and preserves table formatting, but vision-based extraction increases ingestion costs and adds compute latency.
* **Interview-Ready Answer:** "I handle tables by isolating them using layout-aware parsers like Unstructured and converting them to Markdown format. I then generate a textual summary of the table using a fast LLM, vectorize the summary for similarity search, and link it to the raw Markdown table so that if the summary is matched, the LLM receives the full table grid."

---

### Q5: How do you handle PDFs with poor formatting?

* **Definition:** Poorly formatted PDFs include scanned pages, multi-column layouts, mixed-font texts, and documents with embedded figures or mathematical equations.
* **Practical Solution:** 
  1. Run the file through an OCR engine (like **Tesseract** or vision-based models like **Nougat/Grobid**).
  2. Use a layout parser to segment the page into logical columns, processing left-to-right columns sequentially to prevent mixed reading paths.
  3. Clean text noise (broken characters, stray hyphenations, or bad unicode characters) using regex patterns and clean text mapping dictionaries.
* **Tradeoff:** Layout-aware parsing is computationally expensive (requiring GPUs) and slow, but it is necessary to prevent parsing gibberish text that breaks retrieval embeddings.
* **Interview-Ready Answer:** "To parse poorly formatted PDFs, I run a layout-aware vision engine like Nougat or Grobid to segment pages into logical columns and extract math equations as LaTeX. This preserves the sequential reading order of text columns and formats tables cleanly, avoiding the string corruption that occurs with naive PDF parsers."

---

### Q6: How do you handle images/scanned documents?

* **Definition:** Image ingestion processes pictorial data (diagrams, flowcharts, infographics) and extracts the semantic information for text-based retrieval indices.
* **Practical Solution:** I implement a two-pronged ingestion strategy:
  1. **Optical Character Recognition (OCR):** For scanned document pages, run OCR (like PaddleOCR) to extract text blocks.
  2. **Visual Captioning (Multimodal RAG):** For diagrams and charts, feed the image to a multimodal model (like Claude 3.5 Sonnet) with a prompt directing it to write a detailed textual summary of the visual data. Vectorize this text summary and store it in the database with a link to the original image URI.
* **Tradeoff:** Capturing images using multimodal models captures rich information (like flowcharts) but increases ingestion costs by 10x compared to simple text OCR.
* **Interview-Ready Answer:** "For scanned text, I run high-performance OCR engines like PaddleOCR. For diagrams, charts, and flowcharts, I pass the images to a multimodal LLM to write detailed textual descriptions, vectorize those descriptions, and store them in the vector database with a reference link to the source image."

---

### Q7: How do you handle duplicate documents?

* **Definition:** Duplicate document handling identifies identical or highly similar files to prevent indexing redundant vectors, which wastes storage and clutters search results.
* **Practical Solution:**
  1. **Exact Duplicates:** Calculate a cryptographic hash (e.g., MD5) of the raw text during ingestion. If the hash exists in our document database, skip processing.
  2. **Near Duplicates:** Use **MinHash LSH (Locality Sensitive Hashing)** to compute Jaccard similarity scores between documents. If two documents have a similarity $>95\%$, they are flagged as near-duplicates, and we retain only the most recently updated version.
* **Tradeoff:** LSH deduplication prevents vector database bloat and ensures search results are diverse, but adds a hash-matching step during ingestion.
* **Interview-Ready Answer:** "I prevent document duplication by applying exact MD5 hashing for identical files and using MinHash LSH to detect near-duplicates. If a file has a Jaccard similarity of over 95% to an existing document, we flag it and only index the version with the most recent modification timestamp."

---

### Q8: How do you update embeddings when documents change?

* **Definition:** Embedding updates sync the vector database index when source files are edited, added, or deleted, ensuring the retrieval index stays fresh without rebuilding the entire database.
* **Practical Solution:** I implement an incremental pipeline using a **Document State Map**:
  1. Each document is tracked in a relational database with columns: `document_id`, `file_hash`, and `vector_ids`.
  2. When a file is updated, calculate its new hash. If the hash differs, query the vector database using the document's historical `vector_ids` to delete the old vectors.
  3. Re-parse, chunk, and embed the updated file, writing the new vectors to the database and saving the new `vector_ids` to the state map.
* **Tradeoff:** Incremental updates keep search results fresh without needing database rebuilds, but require managing a state database linking files to vector IDs.
* **Interview-Ready Answer:** "I run incremental updates by tracking document states using a metadata mapping database. When a file changes, we compute its new hash, identify its old vector IDs, issue a batch delete command to clear those specific vectors from the database, and then chunk and write the updated embeddings in place."

---

### Q9: How do you delete documents from a vector DB?

* **Definition:** Vector deletion is the physical removal of vector records from a database index.
* **Why it matters:** Vector databases are mostly optimized for read/write. Hard deletions can trigger rebuilding of index graphs (like HNSW), which degrades search performance during deletion batches.
* **Practical Solution:** 
  1. During ingestion, every chunk vector is tagged in the metadata with a `document_id` tag.
  2. When a document is deleted, issue a delete query using the metadata filter: `{"document_id": "TARGET_ID"}`.
  3. If using an HNSW index, perform batch deletions during low-traffic periods to allow the database to re-balance the graph index asynchronously without impacting user search latency.
* **Tradeoff:** Real-time deletions ensure instant data hygiene but trigger continuous graph re-indexing compute overhead. Soft deletions (tagging vectors as `"is_deleted": true` and filtering them out at search time) are faster, but waste storage space.
* **Interview-Ready Answer:** "To delete documents, I tag all vectors with a parent `document_id` in their metadata. When a file is deleted, we issue a delete query filtered by that ID. For high-throughput systems, I run these deletions in batches during low-traffic windows to prevent re-indexing operations from impacting user search latency."

---

### Topic 10: How do you implement access control?

* **Definition:** Access control in RAG restricts document retrieval to only the chunks that the active user is explicitly authorized to view, matching their security clearance or group permissions.
* **Practical Solution:** 
  1. During ingestion, extract the Access Control List (ACL) from the document properties (e.g., `["hr-group", "executive-team"]`).
  2. Save this list in the metadata of every chunk vector as an array parameter: `allowed_groups`.
  3. At query time, retrieve the active user's roles from the session JWT.
  4. Pass these roles as a strict pre-filter in the vector database query (e.g., using pgvector's SQL conditions or Pinecone's metadata query filters).
* **Tradeoff:** Pre-filtering metadata guarantees complete security and ensures the LLM never sees unauthorized data, but it requires syncing ACL changes to vector metadata in real-time.
* **Interview-Ready Answer:** "I implement document security by saving Access Control Lists directly in the chunk metadata as an array. At query time, the orchestrator retrieves the user's authenticated groups from their JWT token and applies them as a strict pre-filtering constraint on the vector database query, ensuring unauthorized documents are excluded before vector similarity is computed."
