# Chapter 8: LangChain, RAG & MCP Concepts — "Advanced AI Architectures"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **LangChain** is a framework for chaining LLM calls. **RAG** (Retrieval-Augmented Generation) feeds external document context to LLMs. **MCP** (Model Context Protocol) is an open standard for connecting LLMs to data sources. |
| **Why It Exists** | Writing custom prompt-routing logic is verbose. LLMs have knowledge cut-offs; RAG injects real-time context. MCP standardizes client-server communication using JSON-RPC. |
| **Where It Is Used** | Production AI agents, chatbot search systems, vector database retrievers, and tool integrations (like connecting IDEs to databases). |
| **Real-World Analogy** | **LangChain:** An assembly chain. A prompt goes in, passes through a formatter (Formatter machine), hits the LLM (Writer machine), and goes to a parser (Sorter machine). **RAG:** A student taking an open-book exam. Instead of answering from memory, the student searches the textbook index (vector lookup) and cites the pages. **MCP:** A USB plug standard. Any phone (LLM client) can connect to any charger (MCP tool server) because they use the same port schema. |

---

## Think of It This Way

### 1. LangChain: The Runnables (LCEL)
LangChain uses **LCEL** (LangChain Expression Language) to chain blocks. Under the hood, this uses Python's operator overloading (specifically the `__or__` method) to pipe components together like a unix pipeline:
`chain = prompt | model | parser`

This compiles to:
`parser(model(prompt(input)))`

All elements in this chain inherit from a base class called `Runnable`, which defines standard methods like `.invoke()`, `.batch()`, and `.stream()`.

### 2. RAG: Cosine Similarity
In RAG, we convert text queries into vectors of floating-point numbers (embeddings).
To find which document matches a user query, we calculate the **Cosine Similarity** between the query vector and the document vectors. Cosine similarity measures the angle between two vectors in multi-dimensional space. If they point in the same direction, similarity is `1.0`; if they are perpendicular, similarity is `0.0`.

---

## Step 1: Cosine Similarity in pure NumPy

Here is how cosine similarity works mathematically under the hood using NumPy:

```python
import numpy as np

def cosine_similarity(v1, v2):
    # Cosine Similarity = (A · B) / (||A|| * ||B||)
    dot_product = np.dot(v1, v2)
    norm_v1 = np.linalg.norm(v1)
    norm_v2 = np.linalg.norm(v2)
    return dot_product / (norm_v1 * norm_v2)

# Simple embedding vectors
query = np.array([0.1, 0.9, 0.0])
doc1  = np.array([0.15, 0.85, 0.05]) # similar
doc2  = np.array([0.9, 0.1, 0.0])    # opposite

print(cosine_similarity(query, doc1)) # ~0.99 (High match!)
print(cosine_similarity(query, doc2)) # ~0.20 (Low match)
```

---

## Step 2: The Model Context Protocol (MCP) JSON-RPC Loop

MCP defines a JSON-RPC 2.0 protocol over standard input/output (`stdin`/`stdout`). Here is how an MCP server declares tools dynamically:

```python
import json
import sys

# Conceptual MCP server loop
def read_input():
    # Reads JSON-RPC request from stdin
    line = sys.stdin.readline()
    if not line:
        return None
    return json.loads(line)

def write_output(response):
    # Writes JSON-RPC response to stdout
    sys.stdout.write(json.dumps(response) + "\n")
    sys.stdout.flush()

def handle_request():
    request = read_input()
    if not request:
        return
    
    method = request.get("method")
    # Exposing tools list
    if method == "tools/list":
        write_output({
            "jsonrpc": "2.0",
            "id": request.get("id"),
            "result": {
                "tools": [
                    {
                        "name": "calc_add",
                        "description": "Add two numbers",
                        "inputSchema": {
                            "type": "object",
                            "properties": {
                                "a": {"type": "number"},
                                "b": {"type": "number"}
                            }
                        }
                    }
                ]
            }
        })
```

---

## Code Examples

### Easy Examples
```python
# 1. Custom Text Chunking (Sliding Window)
# Prepares documents for RAG indexing
def chunk_text(text, chunk_size=100, overlap=20):
    words = text.split()
    chunks = []
    i = 0
    while i < len(words):
        chunk = " ".join(words[i:i + chunk_size])
        chunks.append(chunk)
        i += chunk_size - overlap
    return chunks

text_data = "The quick brown fox jumps over the lazy dog " * 50
print(len(chunk_text(text_data, chunk_size=10, overlap=3)))
```

### Medium Examples
```python
# 2. Overloading __or__ to create a custom Pipe runnable (like LCEL)
class RunnablePipe:
    def __init__(self, func):
        self.func = func

    def __or__(self, other):
        # Pipes this runnable's output into the next runnable's input
        return RunnablePipe(lambda x: other.invoke(self.func(x)))

    def invoke(self, input_data):
        return self.func(input_data)

# Define pipeline components
p1 = RunnablePipe(lambda name: f"Hello {name}")
p2 = RunnablePipe(lambda text: text.upper())

chain = p1 | p2
print(chain.invoke("Alice")) # "HELLO ALICE"
```

### Advanced Examples
```python
# 3. Vector Search Index in pure NumPy
class DenseIndex:
    def __init__(self, dimension):
        self.dimension = dimension
        self.vectors = []
        self.metadata = []

    def add(self, vector, meta):
        # Store as normalized vector
        norm = np.linalg.norm(vector)
        self.vectors.append(vector / norm)
        self.metadata.append(meta)

    def search(self, query_vector, top_k=1):
        if not self.vectors:
            return []
        
        # Convert list of vectors to a single 2D NumPy Matrix
        matrix = np.array(self.vectors) # Shape: (num_docs, dimension)
        q = query_vector / np.linalg.norm(query_vector)
        
        # Matrix multiplication computes cosine similarity for ALL documents in one step!
        similarities = np.dot(matrix, q)
        
        # Get top matching indices
        top_indices = np.argsort(similarities)[::-1][:top_k]
        
        return [(self.metadata[idx], similarities[idx]) for idx in top_indices]

index = DenseIndex(3)
index.add(np.array([0.1, 0.9, 0.0]), "Document A")
index.add(np.array([0.8, 0.2, 0.0]), "Document B")

print(index.search(np.array([0.15, 0.85, 0.0]), top_k=1))
# [('Document A', 0.9986)] (Fast multi-document vector search!)
```

---

## Common Mistakes & Edge Cases

### 1. Matrix operations vs loops in vector databases
When running similarity checks across thousands of documents, beginners often write a `for` loop to compute `cosine_similarity(query, doc)` one-by-one. This is extremely slow. 
* **Fix:** Convert all document vectors into a single 2D NumPy array matrix and use a single matrix dot product (`np.dot(matrix, query)`). NumPy offloads this loop to optimized C/Fortran math libraries, executing it instantly.

---

## Interview Questions (Top 5)

**Q1: How does LangChain Expression Language (LCEL) use Python's operator overloading to build execution chains?**
> LCEL overrides the binary OR operator (`|`) by implementing the `__or__(self, other)` magic method in its base `Runnable` class. When Python compiles `chain = prompt | model`, it runs `prompt.__or__(model)`, which returns a new composite Runnable object. When `.invoke()` is called on the chain, it executes the components sequentially, passing the output of the first as the input of the next.

**Q2: Write the mathematical formula for Cosine Similarity and explain how you implement it efficiently using NumPy.**
> Cosine Similarity is: $Sim(A, B) = \frac{A \cdot B}{\|A\| \|B\|}$. 
> In NumPy, this is calculated as `np.dot(A, B) / (np.linalg.norm(A) * np.linalg.norm(B))`. 
> To do this efficiently at scale, you normalize all document vectors in advance (dividing each by its norm). You can then calculate the cosine similarity for all documents in a single step using matrix-vector multiplication: `np.dot(document_matrix, query_vector)`.

**Q3: What is the Model Context Protocol (MCP) and how does it transport data between LLMs and tool servers?**
> MCP is an open-standard protocol that allows client applications (like AI-native IDEs) to securely discover and invoke tools, prompts, and resources from external servers. It uses JSON-RPC 2.0 messages formatted as text lines, transported over standard input/output (`stdin`/`stdout`) streams or SSE (Server-Sent Events) HTTP connections.

**Q4: In RAG pipelines, what is the role of a Cross-Encoder compared to a Bi-Encoder?**
> * **Bi-Encoder (Retriever):** Generates separate vector embeddings for the query and the documents. Similarity is checked using fast vector dot products. This is fast but less accurate because the query and document do not interact during embedding generation.
> * **Cross-Encoder (Reranker):** Takes the query and a retrieved document together as a single input sequence and passes it through the model. It performs self-attention across both texts, giving a highly accurate similarity score. Because it is computationally expensive, it is only used to rerank the top-10 retrieved documents.

**Q5: What is RunnablePassthrough in LangChain and when do you use it?**
> `RunnablePassthrough` is a component in LCEL used to pass inputs unchanged or with added keys. It is typically used in dict-mapping steps before a prompt is rendered, allowing the pipeline to pass the raw user query down to the next stage while concurrently looking up and injecting the RAG context:
> `{"context": retriever, "question": RunnablePassthrough()}`.

---

*← [Previous: Chapter 7 (ML Pipelines & Agents)](07_Python_ML_Pipelines_Agents.md) | [Next: Chapter 9 (Production Python Best Practices) →](09_Production_Python_Best_Practices.md)*
