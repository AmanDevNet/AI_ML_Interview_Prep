# 📘 Phase 1 Chapter 2: Alignment & Tuning

This chapter covers how base pre-trained models are adapted to follow instructions, behave helpfully, and align with human safety preferences.

---

## 📋 Topics Covered
9. [Topic 9: Fine-Tuning (SFT)](#topic-9-fine-tuning-sft)
10. [Topic 10: Instruction Tuning](#topic-10-instruction-tuning)
11. [Topic 11: RLHF (Reinforcement Learning from Human Feedback)](#topic-11-rlhf-reinforcement-learning-from-human-feedback)

---

### Topic 9: Fine-Tuning (SFT)

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** Supervised Fine-Tuning (SFT) is the process of taking a pre-trained model and training it further on a smaller, curated dataset of input-output pairs (using supervised learning) to adapt its behavior to a specific task or style.
* **Why it exists:** Pretrained base models are excellent at completing text patterns but do not know how to act as conversational assistants or extract specific data formats. SFT updates model weights to enforce target input-output formatting and specialized domain styles.
* **How it works:**
  1. Prepare a high-quality dataset of prompt-response pairs: $\{(x_1, y_1), (x_2, y_2), \dots\}$.
  2. Feed the prompt ($x$) to the model and calculate the model's output distribution for the response ($y$).
  3. Compute loss *only* on the tokens of the response ($y$) using cross-entropy, ignoring the prompt ($x$) tokens (masked loss).
  4. Perform backpropagation to adjust the model's weights using a small learning rate (typically 10x smaller than pretraining).
* **Simple Analogy:** Pretraining is like general school education where you read hundreds of textbooks. SFT is like hired training for a specific job (e.g., customer service agent) where you are given a manual of exact scripts and customer scenarios to practice.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Fine-tuning an open-source model (like Llama 3 8B) on a company's internal customer support logs to make the model sound like an official brand representative.
* **Engineering Tradeoffs:** Full Parameter Fine-Tuning vs. Parameter-Efficient Fine-Tuning (PEFT/LoRA). Full fine-tuning updates all weights, giving maximum capability but requiring huge VRAM and GPU counts (and risking catastrophic forgetting). PEFT (like LoRA) freezes base weights and trains tiny adapter matrices, reducing GPU memory by up to 90% with almost no loss in quality.
* **System Design Relevance:** SFT alters the weights of the model. This means that to serve a fine-tuned model, you must deploy a separate copy of the model parameters, which increases hosting costs. (PEFT/LoRA adapters mitigate this by allowing you to dynamically hot-swap small adapter weights on top of a single shared base model at runtime).

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Supervised Fine-Tuning is the process of modifying a model's weights by training it on labeled prompt-response pairs. It adapts a general base model's behavior, tone, and format to match a target downstream task while keeping compute requirements much lower than pretraining."
* **Common Candidate Mistakes:** Fine-tuning a model to teach it massive amounts of new facts (fine-tuning is great for learning formatting and style, but poor and prone to hallucination for learning concrete new facts; RAG is preferred for factual updates).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is LoRA (Low-Rank Adaptation), and how does it optimize SFT?
    * **Answer:** LoRA freezes the original pretrained model weights and decomposes the weight update matrix ($\Delta W$) into two low-rank matrices ($A$ and $B$, where $\Delta W = B \times A$). By choosing a low rank $r$ (e.g., 8 or 16), we drastically reduce the number of trainable parameters (often by >99%), allowing us to perform fine-tuning on a single consumer GPU instead of a massive cluster.
    * **Follow-up:** How does rank $r$ affect model behavior? (Answer: A higher rank $r$ allows the model to learn more complex behaviors but increases memory footprint and the risk of overfitting. A lower rank acts as a regularizer, restricting the adapter to learning styling/formatting changes).

---

### Topic 10: Instruction Tuning

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** Instruction tuning is a specific form of Supervised Fine-Tuning where the training pairs are explicitly formatted as multi-turn instructions, tasks, and conversational system dialogs, forcing the model to behave as an interactive assistant.
* **Why it exists:** Pretrained base models will often just echo the prompt or continue the text instead of answering. For example, if you prompt a base model with "Write a poem about a tree," it might reply with "Write a poem about a flower." Instruction tuning teaches the model to understand the distinction between context and commands.
* **How it works:**
  1. Datasets are formatted using standard chat markup templates (e.g., ChatML or Llama 3 special tokens like `<|start_header_id|>user<|end_header_id|>`).
  2. Prompts contain explicit instructions, constraint guidelines, and optional system prompts.
  3. The model is trained on a wide variety of tasks (summarization, coding, roleplay, extraction) to ensure it generalizes to following any user instruction rather than just memorizing a single task.
* **Simple Analogy:** Instruction tuning is like teaching a child the difference between repeating a question you asked ("Can you clean your room?") and actually performing the action requested.

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Converting Llama-3-8B (Base) to Llama-3-8B-Instruct. This enables developers to use the model out-of-the-box for interactive chatbots and structured agent tools.
* **Engineering Tradeoffs:** Diversity vs. Performance. Training on too narrow of an instruction set (e.g., only customer support chats) makes the model excellent at that task but ruins its ability to write code or follow general formatting rules. A diverse dataset (like ShareGPT) keeps the model versatile but slightly reduces peak specialized performance.
* **System Design Relevance:** The parsing of instruction tokens must be identical during training and inference. If the production application omits the special chat markup tags (e.g., `<|im_start|>`), the model will fail to recognize where the user's prompt ends and where it should start generating its answer.

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "Instruction tuning is a subset of SFT where the training data consists of explicit instructions and target task completions. It trains the model to act as a helpful conversational agent that executes directives rather than simply completing text."
* **Common Candidate Mistakes:** Believing instruction tuning is a separate training paradigm from SFT (it is SFT, just with instruction-structured datasets).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is catastrophic forgetting in the context of instruction tuning?
    * **Answer:** Catastrophic forgetting occurs when a model is fine-tuned too aggressively on a specific task or style, causing it to lose or degrade its general reasoning, coding, or language capabilities that it acquired during pretraining.
    * **Follow-up:** How do you prevent catastrophic forgetting? (Answer: We can mix a small percentage of general pretraining data or general instruction-following datasets back into the fine-tuning data, or use parameter-efficient methods like LoRA with a low rank).

---

### Topic 11: RLHF (Reinforcement Learning from Human Feedback)

#### 1. Core Mechanics (Definition, Why, How, Analogy)
* **Definition:** RLHF is an alignment methodology that uses reinforcement learning (RL) optimization to align an LLM's outputs with human preferences regarding helpfulness, accuracy, safety, and harmlessness.
* **Why it exists:** SFT alone is limited because it is hard for humans to write perfect gold-standard answers for every scenario. It is much easier for humans to look at two model-generated answers and select which one is better. RLHF leverages these comparative preferences to guide model behavior.
* **How it works:**
  1. **Reward Model Training:** Collect human feedback where annotators rank multiple completions for the same prompt. Train a secondary neural network (the Reward Model) to output a scalar score representing how much humans prefer a given response.
  2. **RL Optimization (PPO):** Use an RL algorithm (like Proximal Policy Optimization) to adjust the LLM's weights. The LLM acts as the policy, generating responses, and the Reward Model acts as the critic, giving feedback. A KL-divergence penalty is added to prevent the LLM from drifting too far from the original SFT model.
  3. **Direct Preference Optimization (DPO):** A modern alternative to RLHF that bypasses training a separate reward model. DPO mathematically reformulates the objective to optimize the LLM's weights directly on the comparison dataset using binary cross-entropy loss.
* **Simple Analogy:** SFT is like showing an art student pictures of master paintings. RLHF is like a teacher standing next to the student, looking at their drafts, and saying "This draft is better because the lighting is realistic, keep doing that."

#### 2. Production & System Dynamics (Real-world, Tradeoffs, Design)
* **Real-World Example:** Safety guardrails. RLHF is the primary tool used by OpenAI and Anthropic to train models to refuse harmful queries (e.g., refusing to write malware) by rating safe refusals higher than successful completions of harmful requests.
* **Engineering Tradeoffs:** Alignment vs. Capabilities. Over-aligning a model using RLHF can make it overly cautious, resulting in "sycophancy" (agreeing with user errors) or "refusal behavior" where it rejects completely benign prompts that contain sensitive keywords (e.g., refusing to summarize a news article about a historical war).
* **System Design Relevance:** Traditional RLHF (PPO) is highly complex, requiring loading up to four separate models into GPU memory simultaneously (the Actor LLM, the Reference LLM, the Critic/Reward model, and the Value model), which demands massive GPU clusters. DPO has largely replaced PPO in production pipelines because it only requires two models (Actor and Reference).

#### 3. Interview Readiness (Direct Answers, Mistakes, Q&As)
* **Interview-Ready Answer:** "RLHF aligns model generation with human preferences using pairwise comparison rankings. It uses these rankings to either train a reward model that guides the LLM via reinforcement learning (PPO), or optimizes the model directly on preferences using Direct Preference Optimization (DPO)."
* **Common Candidate Mistakes:** Assuming RLHF is used to teach models new vocabulary or facts (it is purely used to change the model's preference distribution over behaviors it already knows).
* **Top Interview Q&As & Follow-ups:**
  * **Question 1:** What is the difference between PPO-based RLHF and Direct Preference Optimization (DPO)?
    * **Answer:** PPO-based RLHF requires training an independent Reward Model, which is then used as a reward signal inside a complex actor-critic reinforcement learning loop. DPO mathematically demonstrates that the reward model step can be bypassed. It directly optimizes the LLM parameters on preference pairs (chosen/rejected) by calculating the log ratio of the current model's probabilities vs. a frozen reference model. DPO is simpler, more stable, and requires significantly less GPU memory.
    * **Follow-up:** Why do we need a KL-divergence penalty in RLHF and DPO? (Answer: Without a Kullback-Leibler (KL) penalty, the model will exploit the reward system, finding weird gibberish token sequences that score highly in the reward model but are unreadable to humans—a phenomenon known as reward hacking).
