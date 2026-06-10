# 📓 Phase 5 Chapter 1: Ingestion & Indexing Code

This chapter contains modular, clean Python pseudocode demonstrating the algorithmic mechanics of parsing, splitting, embedding, indexing, and searching document data in production.

---

## 📋 Implementations Covered
1. [Basic RAG Pipeline Pseudocode](#1-basic-rag-pipeline-pseudocode)
2. [Document Loader](#2-document-loader)
3. [Text Splitter](#3-text-splitter)
4. [Embedding Creation](#4-embedding-creation)
5. [Vector DB Insertion](#5-vector-db-insertion)
6. [Similarity Search (NumPy)](#6-similarity-search-numpy)
7. [Retriever Class](#7-retriever-class)

---

### 1. Basic RAG Pipeline Pseudocode
This snippet outlines the end-to-end flow of a RAG query: initializing components, running retrieval, constructing prompts, and querying the generator.

```python
from typing import List, Dict, Any

class RAGPipeline:
    def __init__(self, retriever: Any, generator: Any, prompt_template: str):
        self.retriever = retriever
        self.generator = generator
        self.prompt_template = prompt_template

    def run(self, query: str, user_metadata: Dict[str, Any]) -> str:
        # 1. Retrieve relevant text chunks using metadata filters (e.g. ACLs)
        retrieved_chunks: List[Dict[str, Any]] = self.retriever.retrieve(
            query=query, 
            filters=user_metadata, 
            limit=5
        )
        
        # 2. Extract and format the text contexts from chunks
        context_str = "\n\n".join(
            f"[Doc {chunk['id']}] {chunk['text']}" 
            for chunk in retrieved_chunks
        )
        
        # 3. Assemble the prompt by formatting the template
        prompt = self.prompt_template.format(
            context=context_str, 
            query=query
        )
        
        # 4. Generate the final answer using the LLM client
        response: str = self.generator.generate(prompt)
        return response
```
* **Framework Relation:** Maps to LangChain's `RetrievalQA` chain or LlamaIndex's `QueryEngine`.

---

### 2. Document Loader
Loads files from a target directory, checks cryptographic hashes to prevent re-processing duplicate data, and parses PDF files.

```python
import os
import hashlib
from typing import List, Dict, Any

class LocalDocumentLoader:
    def __init__(self, directory_path: str, processed_hashes_db: Dict[str, str]):
        self.directory_path = directory_path
        self.processed_db = processed_hashes_db # Maps filepath -> md5_hash

    def calculate_md5(self, file_path: str) -> str:
        hash_md5 = hashlib.md5()
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_md5.update(chunk)
        return hash_md5.hexdigest()

    def load_documents(self) -> List[Dict[str, Any]]:
        documents = []
        for filename in os.listdir(self.directory_path):
            if not filename.endswith(".txt"): # Simple text parser example
                continue
            file_path = os.path.join(self.directory_path, filename)
            
            # Deduplication check
            current_hash = self.calculate_md5(file_path)
            if self.processed_db.get(file_path) == current_hash:
                print(f"Skipping unmodified file: {filename}")
                continue
                
            with open(file_path, "r", encoding="utf-8") as f:
                content = f.read()
                
            documents.append({
                "source": filename,
                "text": content,
                "hash": current_hash
            })
            # Update state database
            self.processed_db[file_path] = current_hash
            
        return documents
```
* **Framework Relation:** Serves as the raw implementation behind LangChain's `DirectoryLoader` or LlamaIndex's `SimpleDirectoryReader`.

---

### 3. Text Splitter
A custom implementation of recursive character splitting that prioritizes paragraph breaks, sentence breaks, and spaces, maintaining character bounds.

```python
from typing import List

class RecursiveTextSplitter:
    def __init__(self, chunk_size: int = 500, chunk_overlap: int = 50):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        self.separators = ["\n\n", "\n", " ", ""]

    def split_text(self, text: str) -> List[str]:
        return self._recursive_split(text, self.separators)

    def _recursive_split(self, text: str, separators: List[str]) -> List[str]:
        if len(text) <= self.chunk_size:
            return [text]
            
        # Select active separator
        separator = separators[0]
        next_separators = separators[1:]
        
        # Split text into pieces using active separator
        if separator == "":
            splits = list(text) # Character split fallback
        else:
            splits = text.split(separator)
            
        chunks = []
        current_chunk = ""
        
        for split in splits:
            # Check if split fits in current chunk
            if len(current_chunk) + len(split) + len(separator) <= self.chunk_size:
                current_chunk = current_chunk + (separator if current_chunk else "") + split
            else:
                # Flush full chunk
                if current_chunk:
                    chunks.append(current_chunk)
                # Handle edge case where a single split exceeds chunk_size
                if len(split) > self.chunk_size:
                    if next_separators:
                        chunks.extend(self._recursive_split(split, next_separators))
                    else:
                        chunks.append(split)
                    current_chunk = ""
                else:
                    # Initialize next chunk, adding overlap tokens if available
                    overlap_start = max(0, len(current_chunk) - self.chunk_overlap)
                    current_chunk = current_chunk[overlap_start:] + (separator if current_chunk else "") + split
                    
        if current_chunk:
            chunks.append(current_chunk)
        return chunks
```
* **Framework Relation:** Under-the-hood logic of `RecursiveCharacterTextSplitter`.

---

### 4. Embedding Creation
Wraps client requests to convert text inputs into dense numerical vectors, handling both remote API (OpenAI) and local HuggingFace inferences.

```python
import openai
from typing import List

class EmbeddingGenerator:
    def __init__(self, api_key: str, model: str = "text-embedding-3-small"):
        self.client = openai.OpenAI(api_key=api_key)
        self.model = model

    def get_embeddings(self, text_chunks: List[str]) -> List[List[float]]:
        # Truncate strings to prevent API tokens limit overflow
        cleaned_chunks = [text.replace("\n", " ") for text in text_chunks]
        
        response = self.client.embeddings.create(
            input=cleaned_chunks,
            model=self.model
        )
        # Extract float vectors ordered by input list index
        return [data.embedding for data in response.data]
```
* **Framework Relation:** Wraps LangChain's `OpenAIEmbeddings` or LlamaIndex's `HuggingFaceEmbedding`.

---

### 5. Vector DB Insertion
Handles batch operations, metadata serialization, and writes document records to a database table index.

```python
from typing import List, Dict, Any

class VectorDBConnector:
    def __init__(self, db_client: Any, collection_name: str):
        self.client = db_client
        self.collection_name = collection_name

    def upsert_chunks(self, chunks: List[Dict[str, Any]], embeddings: List[List[float]]):
        payloads = []
        for i, chunk in enumerate(chunks):
            # Format record payload with clean metadata keys
            payloads.append({
                "id": chunk["id"],
                "vector": embeddings[i],
                "payload": {
                    "text": chunk["text"],
                    "source": chunk["source"],
                    "allowed_groups": chunk.get("allowed_groups", ["public"])
                }
            })
        
        # Batch insert to DB client to minimize network IO calls
        self.client.upsert(
            collection_name=self.collection_name,
            points=payloads
        )
        print(f"Successfully indexed {len(payloads)} chunks.")
```
* **Framework Relation:** Analogous to calling `vector_store.add_documents()` in LangChain.

---

### 6. Similarity Search (NumPy)
Implements cosine similarity and dot product scoring from scratch using vector matrices.

```python
import numpy as np
from typing import List, Dict, Any

class LocalVectorSearch:
    @staticmethod
    def cosine_similarity(query_vector: np.ndarray, doc_vectors: np.ndarray) -> np.ndarray:
        # query_vector shape: (D,) | doc_vectors shape: (N, D)
        dot_products = np.dot(doc_vectors, query_vector)
        
        # Compute magnitudes
        query_norm = np.linalg.norm(query_vector)
        doc_norms = np.linalg.norm(doc_vectors, axis=1)
        
        # Avoid division by zero
        doc_norms[doc_norms == 0] = 1e-9
        
        return dot_products / (query_norm * doc_norms)

    @staticmethod
    def dot_product(query_vector: np.ndarray, doc_vectors: np.ndarray) -> np.ndarray:
        # Fast inner-product projection (assumes vectors are pre-normalized)
        return np.dot(doc_vectors, query_vector)
```
* **Framework Relation:** The underlying math executed inside Pinecone, FAISS, or Qdrant indexes.

---

### 7. Retriever Class
Implements a clean, reusable query coordinator class that parses the query, applies metadata pre-filters, and executes the search.

```python
from typing import List, Dict, Any

class MetadataFilteredRetriever:
    def __init__(self, vector_db_client: Any, embedding_model: Any):
        self.db = vector_db_client
        self.embedding_model = embedding_model

    def retrieve(self, query: str, user_roles: List[str], limit: int = 5) -> List[Dict[str, Any]]:
        # 1. Embed query
        query_vector = self.embedding_model.get_embeddings([query])[0]
        
        # 2. Compile pre-filtering dictionary to restrict scope (Access Control List)
        filter_query = {
            "allowed_groups": {
                "$in": user_roles + ["public"]
            }
        }
        
        # 3. Query the database using namespace partition filtering
        results = self.db.search(
            query_vector=query_vector,
            filter_query=filter_query,
            limit=limit
        )
        
        # 4. Map results back to structured text blocks
        return [
            {
                "id": hit.id,
                "text": hit.payload["text"],
                "source": hit.payload["source"],
                "score": hit.score
            }
            for hit in results
        ]
```
* **Framework Relation:** Represents a custom implementation of `vector_store.as_retriever()`.
