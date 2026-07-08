# Generative AI — Interview Prep

> **Tag: Theory + applied judgment** — interviews test conceptual clarity (how LLMs work, RAG vs fine-tuning) and practical LLM-app design, not math derivations.

## Core Concepts

**Generative vs discriminative:** discriminative models predict labels from inputs P(y|x) (spam or not); generative models learn the data distribution well enough to *produce new samples* (text, images, code).

**How LLMs generate text (the 60-second answer):** text → **tokens** (subword units, ~¾ word each) → the model (a Transformer — see `02_TRANSFORMERS.md`) predicts a probability distribution over the next token → sample one → append → repeat. Everything ChatGPT does is iterated next-token prediction; its "knowledge" is patterns compressed into weights during training.

**Training pipeline of a modern assistant:**
1. **Pretraining** — next-token prediction over massive text corpora (self-supervised, no labels) → a base model that completes text.
2. **Supervised fine-tuning (SFT)** — instruction/response pairs → follows instructions.
3. **Alignment (RLHF / DPO)** — human preference data trains the model toward helpful/harmless outputs.

**Sampling controls:** temperature (0 = greedy/deterministic-ish, ↑ = diverse/risky), top-p (sample from smallest set covering p probability mass), max tokens. Know what turning temperature down does and when (extraction/code = low; brainstorming = higher).

**Context window:** the model's working memory (input + output tokens). Doesn't persist between calls — "memory" in chat apps is engineering (resending history, summarization, retrieval).

**Hallucination:** fluent, confident, false output — the model optimizes plausibility, not truth. Mitigations: RAG (ground in retrieved documents), asking for citations, lower temperature, verification steps, tool use (calculators, search).

## The RAG vs Fine-tuning Decision (most-asked applied question)

| | RAG (Retrieval-Augmented Generation) | Fine-tuning |
|---|---|---|
| What | Retrieve relevant docs → stuff into prompt → answer grounded in them | Further train weights on your data |
| Adds | **Knowledge** (fresh, private data) | **Behavior/style/format** |
| Updates | Instantly (reindex) | Retraining cycle |
| Cost | Cheap (embeddings + vector DB) | GPU training + eval |
| Traceability | Citations possible | Black box |
| Default advice | Start here for knowledge tasks | For tone/format/domain-specific *behavior* |

**RAG pipeline (know cold):** chunk documents → **embed** each chunk (vector representing meaning) → store in **vector DB** → at query time: embed the question → nearest-neighbor search (cosine similarity; ANN indexes like HNSW) → top-k chunks into the prompt → LLM answers from them. Failure points: bad chunking, retrieval misses, context stuffing — evaluation matters (retrieval hit rate vs answer quality separately).

**Prompt engineering (legit interview topic):** clear instructions, few-shot examples, role/system prompts, chain-of-thought ("think step by step" → better reasoning), structured output (JSON schemas), delimiters for untrusted input. **Prompt injection:** malicious text in retrieved/user content hijacking instructions — the security question for LLM apps.

## Other Generative Families (one-liners)

**Diffusion models** (image gen — Stable Diffusion, DALL·E): train to denoise; generate by iteratively denoising random noise, optionally text-conditioned. **GANs:** generator vs discriminator adversarial game — sharp images, unstable training, largely superseded for general image gen. **VAEs:** encode to a latent distribution, decode samples — blurry but principled; the latent-space workhorse inside other systems.

## Most-Asked Interview Questions

1. **How does ChatGPT work?** → tokens, next-token prediction, pretraining→SFT→RLHF, sampling. 90 seconds.
2. **What is a token / why do models have token limits?** Subword units; attention cost grows with sequence length (quadratic — see Transformers file).
3. **RAG vs fine-tuning — when each?** → table; knowledge vs behavior is the crisp line.
4. **Explain a RAG pipeline / what's an embedding, vector DB?** → the 6-step pipeline; embedding = dense meaning-vector; vector DB = fast nearest-neighbor at scale.
5. **Why do LLMs hallucinate? How do you reduce it?** Plausibility-optimizer without truth grounding; RAG, citations, tools, verification, temperature.
6. **Temperature/top-p?** Randomness knobs on the sampling distribution — determinism vs diversity.
7. **What is RLHF?** Preference-based alignment stage: reward model from human comparisons → policy optimization (or direct methods like DPO).
8. **GAN vs diffusion vs VAE?** Adversarial / iterative denoising (current SOTA images) / latent-probabilistic.
9. **How would you build a chatbot over company documents?** RAG pipeline + system prompt + chat history management + eval + injection defenses — a mini system-design answer.
10. **What are agents / tool use?** LLM in a loop: reason → call tools (search, code, APIs) → observe → continue; function-calling as the mechanism.
