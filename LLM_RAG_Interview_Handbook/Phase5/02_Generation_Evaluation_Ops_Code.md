# 📓 Phase 5 Chapter 2: Generation, Evaluation & Ops Code

This chapter contains modular, clean Python pseudocode demonstrating the runtime execution phase of RAG: rerankers, prompt templates, completions, citation mapping, telemetry logs, security checks, and streaming.

---

## 📋 Implementations Covered
8. [Reranker Integration](#8-reranker-integration)
9. [Prompt Construction](#9-prompt-construction)
10. [Answer Generation](#10-answer-generation)
11. [Citation Formatting](#11-citation-formatting)
12. [Evaluation Script](#12-evaluation-script)
13. [Feedback Logging (Telemetry)](#13-feedback-logging-telemetry)
14. [Access-Control & Tenancy Filtering](#14-access-control--tenancy-filtering)
15. [Streaming Response](#15-streaming-response)

---

### 8. Reranker Integration
Implements a Cross-Encoder reranker check to re-sort initial candidate chunks based on query alignment.

```python
import requests
from typing import List, Dict, Any

class CohereReranker:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.api_url = "https://api.cohere.ai/v1/rerank"

    def rerank(self, query: str, documents: List[Dict[str, Any]], top_n: int = 5) -> List[Dict[str, Any]]:
        if not documents:
            return []
            
        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }
        
        # Format payload for Cohere API
        doc_texts = [doc["text"] for doc in documents]
        payload = {
            "model": "rerank-english-v3.0",
            "query": query,
            "documents": doc_texts,
            "top_n": top_n
        }
        
        response = requests.post(self.api_url, json=payload, headers=headers)
        response.raise_for_status()
        reranked_data = response.json()["results"]
        
        sorted_docs = []
        for result in reranked_data:
            index = result["index"]
            original_doc = documents[index]
            original_doc["rerank_score"] = result["relevance_score"]
            sorted_docs.append(original_doc)
            
        return sorted_docs
```
* **Framework Relation:** Matches `cohere.Rerank` or LangChain's `CohereRerank` document compressor.

---

### 9. Prompt Construction
Builds structured prompt payloads, formatting variables and placing the most relevant documents in a U-shape (combating attention decay).

```python
from typing import List, Dict, Any

class PromptConstructor:
    def __init__(self, template: str):
        self.template = template

    def build_u_shaped_prompt(self, query: str, documents: List[Dict[str, Any]]) -> str:
        # Sort documents in a U-shape: [1st, 3rd, 5th ... 4th, 2nd]
        sorted_docs = sorted(documents, key=lambda x: x.get("rerank_score", 0), reverse=True)
        
        u_shaped = [None] * len(sorted_docs)
        left = 0
        right = len(sorted_docs) - 1
        
        for i, doc in enumerate(sorted_docs):
            if i % 2 == 0:
                u_shaped[left] = doc
                left += 1
            else:
                u_shaped[right] = doc
                right -= 1
                
        # Format documents with clear XML tags
        context_blocks = []
        for doc in u_shaped:
            if doc is None:
                continue
            context_blocks.append(
                f'<doc id="{doc["id"]}" source="{doc.get("source", "unknown")}">\n'
                f'{doc["text"]}\n'
                f'</doc>'
            )
            
        context_str = "\n\n".join(context_blocks)
        return self.template.format(context=context_str, query=query)
```
* **Framework Relation:** A custom implementation of `PromptTemplate` formatting.

---

### 10. Answer Generation
Interfaces with the completions API to request grounded answers using system instructions and temperature constraints.

```python
import openai
from typing import Dict, Any

class OpenAIGenerator:
    def __init__(self, api_key: str, model: str = "gpt-4o-mini"):
        self.client = openai.OpenAI(api_key=api_key)
        self.model = model

    def generate(self, prompt: str, system_instruction: str) -> str:
        response = self.client.chat.completions.create(
            model=self.model,
            temperature=0.0, # Force deterministic outputs
            messages=[
                {"role": "system", "content": system_instruction},
                {"role": "user", "content": prompt}
            ]
        )
        return response.choices[0].message.content
```
* **Framework Relation:** Wraps LangChain's `ChatOpenAI` completion call.

---

### 11. Citation Formatting
A regex post-processing parser validating that the LLM's generated inline citations map to valid retrieved documents.

```python
import re
from typing import List, Dict, Any

class CitationValidator:
    @staticmethod
    def validate_and_map_citations(answer: str, retrieved_doc_ids: List[str]) -> Dict[str, Any]:
        # Matches patterns like [doc_123] or [123] or [Doc 123]
        pattern = r"\[[D|d]oc[_\s]?(\w+)\]|\[(\w+)\]"
        matches = re.findall(pattern, answer)
        
        # Extract unique citation IDs from multi-group matches
        found_ids = set()
        for m in matches:
            found_ids.add(m[0] if m[0] else m[1])
            
        valid_citations = []
        invalid_citations = []
        
        for cit_id in found_ids:
            if cit_id in retrieved_doc_ids:
                valid_citations.append(cit_id)
            else:
                invalid_citations.append(cit_id)
                
        # Clean answer by stripping out invalid citations to prevent hallucinated references
        cleaned_answer = answer
        for bad_id in invalid_citations:
            cleaned_answer = re.sub(rf"\[[D|d]oc[_\s]?{bad_id}\]|\[{bad_id}\]", "", cleaned_answer)
            
        return {
            "original_answer": answer,
            "cleaned_answer": cleaned_answer,
            "valid_citations": valid_citations,
            "has_hallucinated_citations": len(invalid_citations) > 0
        }
```
* **Framework Relation:** Custom post-processor parser pipeline step.

---

### 12. Evaluation Script
Calculates simple Faithfulness (Grounding) and Context Recall metrics using LLM evaluation rubrics programmatically.

```python
import json
from typing import List, Dict, Any

class Evaluator:
    def __init__(self, judge_client: Any):
        self.judge = judge_client # OpenAI API wrapper

    def evaluate_groundedness(self, answer: str, context: str) -> float:
        # Prompt judge to check if all answer claims exist in context
        prompt = f"""
        Analyze the claims in the Answer and check if they are supported by the Context.
        Output ONLY a valid JSON object with keys: "claims" (list of strings) and "supported_claims" (list of booleans mapping to the claims).
        
        Context: {context}
        Answer: {answer}
        """
        raw_res = self.judge.generate(prompt, system_instruction="You are a strict evaluation judge.")
        try:
            res = json.loads(raw_res)
            claims = res["claims"]
            supported = res["supported_claims"]
            if not claims:
                return 1.0
            return sum(supported) / len(claims) # Groundedness ratio
        except Exception:
            return 0.0
```
* **Framework Relation:** Serves as the raw implementation of a `FaithfulnessEvaluator` in Ragas.

---

### 13. Feedback Logging (Telemetry)
Asynchronously writes user rating events and conversation telemetry payloads to a storage server database.

```python
import aiohttp
import asyncio
from typing import Dict, Any

class AsyncTelemetryLogger:
    def __init__(self, logging_endpoint: str):
        self.endpoint = logging_endpoint

    async def log_feedback(self, interaction_id: str, rating: int, comments: str = ""):
        payload = {
            "interaction_id": interaction_id,
            "rating": rating, # e.g. 1 for thumbs up, -1 for thumbs down
            "comments": comments
        }
        
        # Async HTTP post to prevent blocking the main generator thread loop
        try:
            async with aiohttp.ClientSession() as session:
                async with session.post(self.endpoint, json=payload, timeout=2.0) as resp:
                    if resp.status == 200:
                        print(f"Feedback logged for: {interaction_id}")
                    else:
                        print(f"Telemetry failed: {resp.status}")
        except asyncio.TimeoutError:
            print("Telemetry endpoint timed out, queuing event locally.")
```
* **Framework Relation:** Custom telemetry logging similar to Phoenix or Langfuse tracing integrations.

---

### 14. Access-Control & Tenancy Filtering
Guarantees multi-tenant database isolation and parses credentials at the API gateway layer before retrieval starts.

```python
from typing import List, Dict, Any

class MultiTenantSecurityGateway:
    def __init__(self, db_connector: Any):
        self.db = db_connector

    def execute_secured_query(self, query_vector: List[float], user_token: Dict[str, Any], limit: int = 5) -> List[Dict[str, Any]]:
        # 1. Strict Tenant ID extraction from verified security token
        tenant_id = user_token.get("tenant_id")
        user_roles = user_token.get("roles", [])
        
        if not tenant_id:
            raise PermissionError("Access denied: Invalid tenant token context.")
            
        # 2. Construct logically isolated database query parameters
        # Restrict scope by applying strict namespace matching at database level
        filter_query = {
            "tenant_id": tenant_id,
            "allowed_groups": {
                "$in": user_roles + ["public"]
            }
        }
        
        # 3. DB connector executes query restricted inside tenant boundary
        results = self.db.query(
            vector=query_vector,
            namespace=tenant_id, # Tenant namespace isolation
            filter=filter_query,
            limit=limit
        )
        return results
```
* **Framework Relation:** Security middleware layer for production vector indexes.

---

### 15. Streaming Response
Asynchronously yields generated tokens from the model chunk-by-chunk to the calling client for real-time frontend rendering.

```python
import openai
import asyncio
from typing import AsyncGenerator

class AsyncStreamGenerator:
    def __init__(self, api_key: str, model: str = "gpt-4o-mini"):
        self.client = openai.AsyncOpenAI(api_key=api_key)
        self.model = model

    async def generate_stream(self, prompt: str) -> AsyncGenerator[str, None]:
        response = await self.client.chat.completions.create(
            model=self.model,
            temperature=0.0,
            messages=[{"role": "user", "content": prompt}],
            stream=True # Enable streaming token delivery
        )
        
        async for chunk in response:
            delta = chunk.choices[0].delta
            # Yield next character token as it arrives from the API gateway
            if hasattr(delta, "content") and delta.content is not None:
                yield delta.content
```
* **Framework Relation:** Serves as the implementation of streaming APIs in FastAPI endpoints.
