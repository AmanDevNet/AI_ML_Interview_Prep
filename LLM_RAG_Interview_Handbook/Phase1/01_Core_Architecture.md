# 📘 Phase 1 Chapter 1: Core Architecture

This chapter covers the fundamental structural blocks of Large Language Models (LLMs) that are frequently tested in engineering screenings and system design loops.

---

## 📋 Topics Covered
1. [Topic 1: What is an LLM?](#topic-1-what-is-an-llm)
2. [Topic 2: Tokens and Tokenization](#topic-2-tokens-and-tokenization)
3. [Topic 3: Vocabulary](#topic-3-vocabulary)
4. [Topic 4: Embeddings](#topic-4-embeddings)
5. [Topic 5: Context Window](#topic-5-context-window)
6. [Topic 6: Transformer Architecture (High Level)](#topic-6-transformer-architecture-high-level)
7. [Topic 7: Attention Mechanism (High Level)](#topic-7-attention-mechanism-high-level)
8. [Topic 8: Pretraining](#topic-8-pretraining)

---

### Topic 1: What is an LLM?

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** A Large Language Model (LLM) is a deep autoregressive neural network parameterized by billions of weights, trained on vast textual datasets to predict the conditional probability distribution of the next token given a sequence of preceding tokens: $P(x_{t} \mid x_{1}, x_{2}, \dots, x_{t-1})$.
* **Why it exists:** Traditional NLP relied on task-specific heuristics, rule engines, or small models (like LSTMs) that failed to generalize. LLMs represent a shift toward unified, zero/few-shot foundations where a single pre-trained network generalizes across text generation, reasoning, classification, and translation.
* **How it works:** 
  1. Input text is parsed into tokens and mapped to dense vector spaces (embeddings).
  2. Embeddings pass through multiple stacked Transformer blocks, processing representations in parallel via self-attention.
  3. The final layer projects the processed representation back to a vocabulary-wide logit vector.
  4. The model computes a Softmax probability distribution over this logit vector to sample the next token.
  5. The chosen token is appended to the input, and the loop repeats (autoregressive decoding).
* **Simple Analogy:** Think of an LLM as an incredibly advanced, context-aware autocomplete engine. It doesn't "know" facts in a human sense; it has studied the patterns of millions of books and web pages and calculates which word is most statistically likely to follow the prompt you gave it.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Powering an automated support assistant. The model accepts customer queries, integrates contextual API data, and generates cohesive replies in natural language.
* **Engineering Tradeoffs:** Model Size vs. Latency/Cost. An 8B parameter model is fast and cheap to host, but has weaker reasoning. A 70B parameter model solves complex tasks but requires high-end multi-GPU clusters (e.g., 8x A100s) and introduces significant token latency.
* **System Design Relevance:** LLMs are highly compute-bound and memory-bound. GPU memory (VRAM) dictates the maximum model size and batch capacity you can host. Servicing these models requires optimizations like KV-caching, quantization, and specialized inference engines (e.g., vLLM, TensorRT-LLM).

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "At its core, an LLM is a massive neural network trained to predict the next token in a sequence. It leverages the self-attention mechanism to build a context-rich mathematical representation of input text and projects this representation onto a vocabulary space to sample the most probable continuation."
* **Common Candidate Mistakes:** Describing the model as a database that "queries its knowledge base" during generation, or claiming that it possesses reasoning capabilities identical to human logic rather than statistical pattern mapping.
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** Why is next-token prediction so effective at performing complex reasoning tasks?
    * **Answer:** By learning to predict the next token over massive web-scale corpora, the model is forced to compress and model the underlying rules of human language, grammar, code logic, and factual structures. Reasoning emerges as a byproduct of learning this deep statistical structure.
    * **Follow-up:** Does this mean a model can reason about a logic puzzle it has never seen before? (Answer: Yes, to a degree, by composing sub-patterns and structural templates it learned during training, though it remains prone to structural failure when the puzzle deviates slightly from distribution).

---

### Topic 2: Tokens and Tokenization

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** Tokenization is the preprocessing step that converts raw string text into discrete numerical IDs (tokens) that can be processed by a neural network.
* **Why it exists:** Neural networks cannot process raw characters or text strings directly. Tokenization bridges the gap by chunking text into sub-word units, balancing vocabulary size against sequence length.
* **How it works:**
  1. Modern tokenizers use algorithm variants like Byte-Pair Encoding (BPE) or WordPiece.
  2. The tokenizer builds a vocabulary of common sub-word units by iteratively merging the most frequent character pairings in a training corpus.
  3. When encoding text, it decomposes words into these learned sub-word units (e.g., "unhelpful" $\rightarrow$ `["un", "helpful"]`).
  4. Each sub-word token is mapped to its corresponding integer index in the vocabulary dictionary.
* **Simple Analogy:** Imagine reading a complex language by breaking down words into prefixes, roots, and suffixes. Instead of spelling out "i-m-p-o-s-s-i-b-l-e" letter-by-letter, you read it as "im-" and "-possible".

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Tokenizer mismatches in API calls. If you tokenize text using the Llama 3 tokenizer but send the IDs to a Mistral model, the model receives incorrect vocabulary mappings, resulting in gibberish output.
* **Engineering Tradeoffs:** Large Vocabulary Size vs. Model Parameter Overhead. A large vocabulary (e.g., Gemma's 256k tokens) represents text in fewer tokens (reducing sequence length), but consumes massive GPU memory in the embedding and final linear projection layers. A smaller vocabulary (e.g., Llama 2's 32k tokens) saves parameters but produces longer token sequences for the same text, increasing computation time.
* **System Design Relevance:** Billing in LLM APIs is calculated per token, not per character or word. Furthermore, tokenization speeds can become a bottleneck in data-intensive extraction pipelines if not parallelized.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Tokenization is the process of splitting text into sub-word units called tokens, using algorithms like BPE, and mapping them to unique vocabulary integers. This balances vocabulary size with sequence length, enabling the model to handle rare and misspelled words without running out of memory."
* **Common Candidate Mistakes:** Assuming 1 token always equals 1 word (1 token is roughly 0.75 words), or forgetting that spaces, punctuation, and capitalizations are encoded as part of the tokens.
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** Why do LLMs struggle with basic spelling questions (e.g., counting the letters in 'strawberry')?
    * **Answer:** LLMs do not see raw characters. The tokenizer groups "strawberry" into a single token block like `[str-aw-berry]`. Because the model processes these chunks as integer IDs, it lacks character-level awareness unless it is explicitly prompted to break down the word character-by-character.
    * **Follow-up:** How can we fix this in an engineering pipeline? (Answer: We can preprocess character-heavy queries by inserting spaces between letters before feeding them to the model, or use character-aware model architectures).

---

### Topic 3: Vocabulary

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** The vocabulary is the fixed set of unique sub-word tokens that the tokenizer can recognize and map to integers.
* **Why it exists:** The model's final projection layer (the LM head) uses a matrix size bound directly to the vocabulary size. A fixed vocabulary is required to keep parameter sizes stable and to apply Softmax normalization over a set number of choices.
* **How it works:**
  1. The vocabulary is determined *before* the model starts training, during the tokenizer build phase.
  2. It contains common words, sub-words, characters, punctuation, and special tokens (e.g., `<|endoftext|>`).
  3. Any input character sequence not directly in the vocabulary is broken down into constituent bytes (fallback) to avoid the Out-Of-Vocabulary (OOV) error that plagued early NLP.
* **Simple Analogy:** The vocabulary is the model's dictionary. If a word is not in the dictionary, the model must break it down into smaller syllables or individual letters that it does recognize to understand it.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Non-English performance. If you prompt an English-centric model (like Llama 2) in Hindi or Russian, it will tokenize the input into hundreds of tiny individual bytes or character tokens, blowing up your context window usage and driving up latency.
* **Engineering Tradeoffs:** Vocabulary Size vs. Speed. Increasing vocabulary size reduces token counts for non-English languages and code, but it increases the size of the final softmax classification layer, causing a noticeable compute overhead at every step of generation.
* **System Design Relevance:** The embedding matrix and the final linear layer together can account for up to 10-30% of a smaller model's parameters. This must be factored into memory footprints when loading models on edge devices.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "A model's vocabulary is the predefined set of all unique tokens it can read and write. It is fixed during the tokenizer training phase, and its size dictates the dimension of the embedding layer and the final output logits matrix."
* **Common Candidate Mistakes:** Thinking the vocabulary can be dynamically expanded after the model is trained, or assuming the model can only generate words that exist in its vocabulary as complete strings.
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What happens if a model encounters an emoji or character that was not in its vocabulary during tokenizer creation?
    * **Answer:** Modern tokenizers use byte-level fallbacks. Instead of throwing an error, the tokenizer breaks the unknown character down into its UTF-8 raw bytes, treating each byte as a separate token. The model then processes those bytes.
    * **Follow-up:** Does this affect model performance? (Answer: Yes, because it increases token count and the model has likely seen very few byte-level patterns for that specific emoji during training, leading to degradation).

---

### Topic 4: Embeddings

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** An embedding is a dense, low-dimensional vector representation of a token in a continuous vector space, where tokens with similar semantic or syntactic meanings are positioned closer to one another.
* **Why it exists:** Computer models cannot calculate mathematical relationships on discrete integer IDs. Mapping tokens to a high-dimensional vector space (e.g., 4096 dimensions) allows the model to calculate semantic similarities and feed continuous representations to transformer blocks.
* **How it works:**
  1. The input token ID indexes a massive lookup matrix (Embedding Matrix $W_e$) of size $\text{Vocab Size} \times d_{\text{model}}$.
  2. This lookup extracts a single row vector corresponding to the token.
  3. Positional encodings (e.g., RoPE) are mathematically added to this token embedding vector to capture the token's position in the sentence.
* **Simple Analogy:** Imagine mapping words on a massive map. Words like "king" and "queen" would be placed very close to each other. Furthermore, the distance and direction from "man" to "woman" would be the same as the distance and direction from "king" to "queen" (vector arithmetic).

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Search engines and semantic retrieval. We use standalone embedding models (like `text-embedding-3-small`) to convert search queries and documents into vectors, and then run vector similarity searches to fetch relevant chunks.
* **Engineering Tradeoffs:** Embedding Dimension Size vs. Quality/Latency. Larger embedding dimensions (e.g., 1536 vs 384) capture richer semantic relationships but require more memory storage in vector databases, slow down search index performance, and increase network serialization overhead.
* **System Design Relevance:** Embeddings are static at the input layer. However, as they pass through the transformer blocks, the attention layers dynamically modify these vectors, shifting their positions in the vector space based on context.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Embeddings are dense vector representations of tokens where semantic meaning is captured by spatial distance. They convert raw discrete token IDs into continuous multidimensional vectors that neural networks can process using vector mathematics."
* **Common Candidate Mistakes:** Confusing *token* embeddings (lookup vectors inside the LLM) with *sentence* embeddings (aggregated single vectors representing a full chunk of text, used in vector databases for RAG).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the difference between static token embeddings and contextual token representations inside the Transformer?
    * **Answer:** Static token embeddings are retrieved from the lookup table and are identical for a token regardless of context (e.g., "bank" in "river bank" vs "money bank"). Contextual representations are the vectors output by the self-attention blocks, which have blended information from surrounding tokens to disambiguate meaning.
    * **Follow-up:** How does the model keep track of word order in the static embedding layer? (Answer: Through positional encodings like Rotary Position Embeddings (RoPE), which encode absolute or relative token positions into the vector dimensions).

---

### Topic 5: Context Window

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** The context window is the maximum length of the input plus output sequence (measured in tokens) that the model can process in a single forward pass.
* **Why it exists:** The self-attention mechanism scales quadratically $O(N^2)$ in time and space with respect to the sequence length. Physical GPU memory constraints limit the maximum sequence length a model can compute.
* **How it works:**
  1. When a prompt is passed to the model, it is tokenized.
  2. The input tokens plus the newly generated tokens must fit within the model's configured limit (e.g., 8k, 32k, or 1M tokens).
  3. Inside the self-attention calculation, an $N \times N$ attention matrix is formed (where $N$ is the sequence length). 
  4. If $N$ exceeds the context window, the model cannot map the attention weights correctly because positional encodings may fail, or the physical GPU memory will overflow (Out of Memory - OOM).
* **Simple Analogy:** The context window is like a person's short-term working memory. If you give them a 500-page book to read and immediately ask a question about Chapter 1, they can only answer if they can hold the details of Chapter 1 in their working memory. If they forget it, they can't reference it.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Passing an entire codebase or API specification into a prompt to debug a function. If the codebase is larger than the model's context window, you must chunk the files and retrieve relevant snippets (RAG) rather than passing everything.
* **Engineering Tradeoffs:** Context Length vs. System Performance. Long-context models (e.g., 128k or 1M context) allow massive prompts, but generating answers inside long contexts degrades latency because the attention matrix computation scales quadratically, and KV cache size increases dramatically, reducing parallel batch capacity.
* **System Design Relevance:** Long contexts require specialized techniques like FlashAttention to optimize memory, RoPE base frequency scaling to extrapolate positions, and KV Cache compression.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "The context window is the maximum sequence length of tokens a model can process in a single forward pass. It is structurally limited by the quadratic scaling of self-attention compute requirements and the physical VRAM memory of the hosting GPUs."
* **Common Candidate Mistakes:** Believing that a model with a 128k context window can retrieve information from the middle of the prompt with 100% accuracy (referred to as the "lost in the middle" phenomenon).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the "Lost in the Middle" phenomenon, and how does it affect long-context design?
    * **Answer:** Research shows LLMs are much better at retrieving information placed at the very beginning or end of their input context than information buried in the middle. When designing prompts for long-context models, critical instructions or reference facts should be placed at the top or bottom of the prompt to maximize extraction accuracy.
    * **Follow-up:** How does KV cache scale with long contexts? (Answer: The KV cache scales linearly with sequence length, batch size, and model dimensions. For a 128k context, the KV cache can easily exceed the size of the model weights themselves, requiring offloading or quantization).

---

### Topic 6: Transformer Architecture High Level

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** The Transformer is a deep learning architecture characterized by stacked layers of multi-head self-attention and position-wise feed-forward networks, designed to process sequential data in parallel without recurrence.
* **Why it exists:** Older sequential architectures like RNNs and LSTMs processed text word-by-word, creating sequential bottlenecks that prevented parallel training on modern GPU clusters. Transformers process the entire sequence at once, allowing massive scaling.
* **How it works:**
  1. **Input Encoding:** Words are tokenized and converted to embeddings, with positional encodings added.
  2. **Encoder Blocks (Optional/Seq2Seq):** Process input context to create a rich, bidirectionally attended representation.
  3. **Decoder Blocks:** Autoregressively generate text. They use *masked* self-attention to prevent the model from peeking at future tokens during training.
  4. **Feed-Forward Networks (FFN):** A set of linear layers applied to each token vector individually to project them to higher-dimensional spaces for non-linear feature mapping.
  5. **Layer Normalization & Residual Connections:** Stabilize gradient flow in deep networks.
* **Simple Analogy:** Think of an RNN as a factory assembly line where each worker has to wait for the previous worker to finish their step. A Transformer is like a team of workers working in a shared digital doc simultaneously, instantly communicating edits to everyone else.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Choosing between an Encoder-only model (like BERT) for lightning-fast text classification, and a Decoder-only model (like GPT-4 or Llama) for conversational generation.
* **Engineering Tradeoffs:** Encoder-Decoder vs. Decoder-only. Encoder-Decoder architectures (like T5) are excellent for translation and summarization, but Decoder-only architectures have proven much easier to scale to hundreds of billions of parameters for generalized reasoning.
* **System Design Relevance:** The FFN layer contains the vast majority of static model parameters (weights). Optimizing this layer (e.g., using Mixture of Experts - MoE, where only a subset of FFN experts are activated per token) dramatically reduces inference compute costs.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "The Transformer is a non-recurrent neural network architecture that processes sequences in parallel. It relies on self-attention blocks to compute dynamic contextual representations of tokens, followed by feed-forward networks to extract high-level features."
* **Common Candidate Mistakes:** Forgetting the distinction between Encoders (which allow bidirectional attention) and Decoders (which use causal masking to restrict attention to preceding tokens).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** Why do modern LLMs (like Llama, GPT, Mistral) almost exclusively use Decoder-only architectures?
    * **Answer:** Decoder-only models are highly efficient for autoregressive generation because they compute the key-value states for the next token incrementally, making pre-training on raw unlabelled text simple. Scaling laws also showed that Decoder-only architectures scale more predictably with massive compute budgets.
    * **Follow-up:** How does causal masking work in the decoder? (Answer: It applies a upper-triangular mask of negative infinity values to the attention logits matrix, which forces the softmax probability of attending to future tokens to evaluate to zero).

---

### Topic 7: Attention Mechanism High Level

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** Self-attention is a mathematical mechanism that computes dynamic weights between all pairs of tokens in a sequence, determining how much context each token should extract from every other token.
* **Why it exists:** Hard-coded mappings or fixed-window associations fail to resolve context-dependent meanings (e.g., "The bank of the river" vs "The bank of the country"). Attention allows tokens to dynamically update their vector values based on the specific context they appear in.
* **How it works:**
  1. For each token's vector, we project three new vectors using learned weight matrices: **Query ($Q$)**, **Key ($K$)**, and **Value ($V$)**.
  2. The Query vector represents what the token is looking for.
  3. The Key vector represents what information the token contains.
  4. The Value vector contains the actual semantic content of the token.
  5. We take the dot product of all Queries and Keys to compute raw similarity scores (attention logits): $Q K^T$.
  6. We scale the scores by $\sqrt{d_k}$ (to prevent vanishing gradients) and apply a Softmax to get probability weights: $\text{Softmax}(\frac{Q K^T}{\sqrt{d_k}})$.
  7. We multiply these weights by the Value vectors to produce the final contextualized token representation: $\text{Attention}(Q, K, V) = \text{Softmax}(\frac{Q K^T}{\sqrt{d_k}}) V$.
* **Simple Analogy:** Imagine a filing cabinet. Your query is the sticky-note description of what folder you want. The key is the label printed on each physical file folder. You match your sticky-note against the labels (dot product) to find the right folder, and then open the folder to read the contents inside (the value).

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Multi-head attention allows a model to concurrently resolve multiple relationships, such as identifying both *who* performed an action and *where* it was performed in a long sentence.
* **Engineering Tradeoffs:** Multi-Head Attention (MHA) vs. Grouped-Query Attention (GQA). MHA assigns separate Q, K, and V heads to every attention head, which provides high capacity but a massive KV cache memory footprint. GQA groups multiple Query heads to share a single Key/Value head group, reducing KV Cache size by up to 8x with minimal loss in model quality.
* **System Design Relevance:** Standard self-attention scales quadratically, making it the primary bottleneck for long-context queries. GPU operations must be optimized (e.g., FlashAttention, which avoids writing the intermediate $N \times N$ attention matrix to slow high-bandwidth GPU memory by keeping it in fast SRAM).

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Self-attention computes query, key, and value vectors for each token, using dot products to calculate similarity weights between queries and keys. These weights are normalized via softmax and used to compute a weighted sum of the values, generating context-aware representations."
* **Common Candidate Mistakes:** Inability to explain the physical mathematical shapes of $Q$, $K$, and $V$ matrices, or failing to explain why the scale factor ($\sqrt{d_k}$) is necessary (prevents Softmax output from pushed into regions of tiny gradients).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the difference between Multi-Head Attention (MHA), Multi-Query Attention (MQA), and Grouped-Query Attention (GQA)?
    * **Answer:** MHA uses unique Query, Key, and Value projections for each head. MQA uses unique Query projections but forces all heads to share a single Key and Value projection, which dramatically reduces KV Cache memory but harms performance. GQA is a compromise where a subset of Query heads share key/value projections (e.g., 8 Queries per 1 Key/Value group), balancing memory savings and accuracy.
    * **Follow-up:** Which of these does Llama 3 use? (Answer: Llama 3 uses Grouped-Query Attention).

---

### Topic 8: Pretraining

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** Pretraining is the initial phase of model training where the model is trained on a massive, unlabeled text corpus (trillions of tokens) using self-supervised objectives, primarily causal language modeling (next-token prediction).
* **Why it exists:** Pretraining builds the foundational representation, world knowledge, grammar rules, and reasoning structures inside the model's weights. Without pretraining, a model cannot understand language patterns or relationships.
* **How it works:**
  1. Terabytes of raw text are collected, filtered, and tokenized.
  2. The model parameters are randomly initialized.
  3. Batches of tokens are fed to the model, and it outputs predictions for the next token.
  4. The model's prediction is compared to the actual next token using a Cross-Entropy Loss function.
  5. Backpropagation computes gradients, and optimizer algorithms (like AdamW) update the model's weights.
  6. This is repeated across trillions of tokens over weeks or months on thousands of interconnected GPUs.
* **Simple Analogy:** Pretraining is like a human child reading every book in a massive library. They aren't trying to pass a specific exam; they are simply absorbing how sentences are structured and learning factual relationships about the world.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Choosing whether to build a domain-specific model (like BioGPT) by pretraining from scratch on medical text, or simply fine-tuning a general-purpose pretrained model (like Llama 3).
* **Engineering Tradeoffs:** Training from Scratch vs. Starting from Pretrained. Pretraining a base model is incredibly expensive (costing millions of dollars in compute) but gives you full control over the vocabulary and domain alignment. Fine-tuning an existing model costs a fraction of the price but inherits the base model's bias and license terms.
* **System Design Relevance:** Pretraining requires massive scale infrastructure. It relies on distributed training strategies: Data Parallelism (splitting data batches across GPUs), Tensor Parallelism (splitting layers across GPUs), and Pipeline Parallelism (splitting different blocks of layers across nodes).

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Pretraining is the self-supervised phase where an LLM is trained on trillions of tokens of unlabeled text to minimize next-token prediction loss. This process instills basic grammar, world facts, and statistical reasoning structures into the network's weights."
* **Common Candidate Mistakes:** Assuming pretraining makes a model helpful or conversational (pretrained base models will often just replicate patterns or repeat the user's prompt rather than answering).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What are Kaplan's and Chinchilla's Scaling Laws, and how do they impact pretraining budgets?
    * **Answer:** Kaplan's scaling laws initially suggested that model parameter size should be scaled faster than data volume. DeepMind's Chinchilla scaling laws corrected this, proving that for optimal compute allocation, parameter size and dataset size should scale in equal proportion. An optimal model should have roughly 20 tokens of training data per 1 model parameter.
    * **Follow-up:** If Chinchilla suggests 20 tokens per parameter, why was Llama 3 8B trained on 15 trillion tokens (nearly 1800 tokens per parameter)? (Answer: Chinchilla laws optimize for *training* compute efficiency. However, in production, we want *inference* compute efficiency. Training a smaller model far past its Chinchilla-optimal point makes it smarter and much cheaper to run in production at scale).
