# 📗 Phase 2 Chapter 5: Generation & Synthesis

This chapter covers the synthesis phase of RAG: how retrieved document chunks are ordered, how prompt templates are constructed, and how we guide the generator to output factually grounded answers with inline citations.

---

## 📋 Topics Covered
26. [Topic 26: Context Construction](#topic-26-context-construction)
27. [Topic 27: Prompt Template for RAG](#topic-27-prompt-template-for-rag)
28. [Topic 28: Citations](#topic-28-citations)
29. [Topic 29: Source Attribution](#topic-29-source-attribution)
30. [Topic 30: Hallucination Reduction](#topic-30-hallucination-reduction)

---

### Topic 26: Context Construction

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Context Construction is the step where retrieved document text chunks are formatted, sorted, and assembled into the context window parameter of the LLM prompt.
* **Why it matters:** Just passing raw text blocks randomly into a prompt leads to poor model reasoning. The structural layout and sequence order of the context documents directly affect what facts the model's attention layers prioritize.
* **How it affects answer quality:** High. If context construction places the most critical answer chunk in the middle of a massive context block, the model's attention mechanism may drop it (Lost in the Middle), resulting in an incomplete or incorrect answer.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Sorting Order vs. Retrieval Rank. Traditional sorting puts documents in descending order of similarity rank. However, to combat attention decay, placing the highest-ranked documents at the extreme beginning and end of the context block (U-shaped sorting) often improves retrieval accuracy.
* **Production Example:** Formatting retrieved chunks with explicit XML tag wrappers:
  ```xml
  <doc id="1" source="policy_doc.pdf">
  Employees are allowed up to 15 days of sick leave per year.
  </doc>
  ```

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Context construction compiles retrieved text chunks into structured context fields within the LLM prompt. We use clean delimiters like XML tags and apply strategic sorting to prevent the model from missing facts buried in the middle of the prompt."
* **Top Q&As:**
  * **Question 1:** How do you structure the prompt to mitigate the "Lost in the Middle" phenomenon when you have 15 retrieved documents?
    * **Answer:** We write a sorting function that distributes documents in a U-shape: placing the top 1st and 2nd most relevant documents at the very beginning of the context, the 3rd and 4th at the very end, and the least relevant documents in the middle. This aligns with the model's natural positional attention bias.
    * **Follow-up:** How do you clean the text before context construction to save token space? (Answer: We compress the text by removing redundant white spaces, stripping out duplicate paragraphs, and occasionally running a lightweight sentence-boundary classifier to drop sentences that do not align with the retrieved target).

---

### Topic 27: Prompt Template for RAG

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** A RAG Prompt Template is a structured text scaffold containing placeholders for system rules, retrieved context, chat history, and the user's current query, dynamically compiled at runtime.
* **Why it exists:** Aligned LLMs require explicit instruction blocks to understand how to interact with retrieved information. The template acts as the boundary container that restricts the model from using external knowledge.
* **How it affects answer quality:** A poorly written template will allow the model to ignore the context and hallucinate from its weights, or refuse to answer when the information is present. A well-designed template clearly defines the rules of grounding.

#### 2. Engineering Dynamics (Tradeoffs & Production examples)
* **Engineering Tradeoffs:** Strict Refusal vs. General Utility. A template that strictly instructs: *"If the answer is not in the context, say 'I don't know'"* reduces hallucinations but can cause false-positive refusals if the answer requires making a simple connection between facts.
* **Production Example:**
  ```text
  System: You are a helpful assistant. Use ONLY the provided context blocks to answer the question.
  
  Context:
  {context_documents}
  
  User Question: {user_query}
  
  Answer:
  ```

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "The prompt template is the instruction wrapper for the LLM. It defines the rules of grounding, formats the retrieved context, and injects the user's query, forcing the model to act as a synthesizer rather than a general writer."
* **Top Q&As:**
  * **Question 1:** What are the key instructions you must include in a production RAG prompt template to ensure reliability?
    * **Answer:** I always include three key instructions:
      1. Direct the model to prioritize the provided context over its own internal knowledge.
      2. Set a clear refusal clause if the context is insufficient (e.g., "Reply with 'I do not have enough information' if the answer cannot be found in the context").
      3. Instruct the model to cite the specific document ID or source name when stating a fact from the text.
    * **Follow-up:** How do you handle formatting constraints (e.g., requiring JSON)? (Answer: We append the JSON schema rules to the bottom of the prompt template and enforce it programmatically using grammar-based sampling tools like Instructor).

---

### Topic 28: Citations

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Citations are explicit references (e.g., `[Doc 1]`, `[source: HR_Manual.pdf]`) generated inline by the LLM, linking specific claims in the output to the corresponding document chunk used to generate that claim.
* **Why it exists:** Citations are the primary mechanism for establishing trust and verify outputs in professional applications (like legal, medical, or financial systems), allowing users to audit the model's claims.
* **How it works:**
  1. The retrieved documents are mapped to unique IDs (e.g., `[1]`, `[2]`).
  2. The prompt template instructs the model: *"You must cite your sources inline using the format [ID] when referencing a document."*
  3. The model generates text, inserting the token sequence matching the citation brackets.
  4. The application parses the brackets and converts them into clickable hyperlinks linking to the source files.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** A medical diagnosis checker that outputs: *"The recommended dosage of Ibuprofen for adults is 400mg every 4-6 hours [Source 3]."* The user can click "[Source 3]" to view the official FDA drug label PDF.
* **Engineering Tradeoffs:** Formatting Adherence vs. Reasoning Latency. Forcing the model to write complex citation strings inline can slightly decrease generation speed and, in weaker models, lead to formatting errors that break the parser.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Citations instruct the model to write inline references linking its claims to the specific IDs of the retrieved documents. This makes the generated answer auditable and allows the frontend to map the response to source hyperlinks."
* **Top Q&As:**
  * **Question 1:** How do you verify that the model's generated citations are actually correct and not hallucinated?
    * **Answer:** We write a post-processing validation parser. The parser extracts all inline citation IDs (e.g., `[3]`) from the generated text and checks them against the list of IDs that were actually retrieved and passed in the context. If the model cites a document ID that was never sent (e.g., citing `[9]` when only 1-5 were sent), the system flags it as a hallucinated citation and falls back to a correction step.
    * **Follow-up:** How do you automate this correction? (Answer: We prompt a fast model with the generated text and the retrieved context, asking it to verify or correct the citation mappings, or use programmatic string matching to align claims to sentences).

---

### Topic 29: Source Attribution

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Source Attribution is the backend mapping process that pairs the LLM's generated citations with actual document URLs, filenames, page numbers, or paragraphs stored in the database metadata.
* **Why it matters:** A citation like `[Doc 3]` is useless to a user unless they can easily view the source. Source attribution resolves these tokens to physical files, ensuring a clean user experience.
* **How it affects answer quality:** It affects the user's perception of quality. By displaying the exact page or paragraph the model read, it builds trust and makes validation simple.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Linking the chatbot output to the exact page of a 500-page manual. The metadata for Chunk 3 contains `{"source_url": "manual.pdf", "page": 42}`. The system renders a PDF viewer opened directly to page 42.
* **Engineering Tradeoffs:** Granularity vs. Ingestion Overhead. Attributing down to the exact paragraph requires complex parsing and metadata tracking during ingestion. Attributing to the general document is simpler but less helpful for long files.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "Source attribution maps the model's generated citation markers back to physical document URLs, pages, or sections stored in the chunk's metadata, enabling direct verification paths."
* **Top Q&As:**
  * **Question 1:** What happens if the model groups facts from multiple documents into a single sentence but only cites one?
    * **Answer:** This represents an attribution gap. We minimize this by instructing the prompt template: *"If a claim draws from multiple documents, you must list all matching citation IDs (e.g., [1][2])."* During post-processing, we parse these concatenated brackets to map multiple source links to that sentence.
    * **Follow-up:** How does chunk overlap affect source attribution? (Answer: If a fact is found in the overlapped region of Chunk 1 and Chunk 2, the model might cite either one. The source attribution system should resolve both to the same parent document URL to avoid confusing the user).

---

### Topic 30: Hallucination Reduction

#### 1. Core Mechanics (Definition, Why, Quality Impact)
* **Definition:** Hallucination Reduction in RAG refers to the combination of prompt constraints, retrieval filtering, and post-generation evaluation designed to minimize the generation of claims not supported by the retrieved context.
* **Why it exists:** Even with retrieved context, LLMs can ignore the data and prioritize their pre-trained weights, leading to false claims. Hallucination reduction is mandatory for deploying RAG in critical business settings.
* **How it works:**
  1. **Prompt Constraint:** Enforcing a strict "grounding instruction" (e.g., *"Do not make claims outside of the provided context"*).
  2. **Low Temperature:** Setting temperature to 0 to eliminate probabilistic sampling variances that can introduce errors.
  3. **Verification Step:** Running a programmatic self-consistency check or using an NLI classifier to verify that the generated output is logically entailed by the retrieved context.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** A pharmaceutical medical lookup bot. If the model suggests a drug dose not explicitly written in the retrieved prescribing information, the safety checker intercepts the response and blocks it.
* **Engineering Tradeoffs:** Output Grounding vs. Conversational Flow. Aggressive hallucination reduction rules can make the model sound highly defensive, repeating "I don't know" for questions that could easily be answered with minor synthesis of the retrieved text.

#### 3. Interview Readiness (Explanation, Q&As, Follow-ups)
* **Interview Explanation:** "We reduce hallucinations in RAG by combining strict prompt grounding rules, setting the sampling temperature to zero to eliminate random variances, and running post-generation verification checks to ensure every claim is entailed by the retrieved sources."
* **Top Q&As:**
  * **Question 1:** How does setting the temperature to 0 reduce hallucinations?
    * **Answer:** Setting temperature to 0 forces the model to perform greedy decoding, choosing the token with the absolute highest probability at every step. This removes the random sampling tail where the model might otherwise select a lower-probability, hallucinated token.
    * **Follow-up:** Can a model still hallucinate at temperature 0? (Answer: Yes. If the prompt is ambiguous, if the retrieved context contains contradictory statements, or if the model's weights have a very strong pre-trained bias toward a specific word pattern, the highest-probability token at temperature 0 can still be incorrect).
