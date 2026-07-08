# AI / ML — Index

## Study Order & Priorities

1. `02_TRANSFORMERS.md` — **Tier 1.** Self-attention is the single most-asked modern ML question; everything else references it.
2. `01_GENERATIVE_AI.md` — **Tier 1.** LLM mechanics + RAG vs fine-tuning = the applied questions every 2025+ interview asks.
3. `03_NLP.md` — **Tier 2.** The representation-evolution narrative (BoW → TF-IDF → Word2Vec → BERT → LLMs) organizes the whole field.
4. `04_COMPUTER_VISION.md` — **Tier 2** (Tier 1 for CV roles). CNN mechanics + transfer learning.

## Fast-Scan Comparisons

**Generative vs discriminative:** produce samples from learned distribution vs predict labels.

**RAG vs fine-tuning:** add knowledge (fresh, citable, cheap) vs add behavior/style (training cycle). Start with RAG for knowledge tasks.

**BERT vs GPT (encoder vs decoder):** bidirectional understanding, masked-LM vs causal generation, next-token. Encoder-decoder (T5) for translation/summarization.

**RNN vs Transformer:** sequential + fading long-range vs parallel + direct attention; O(n) steps vs O(n²) attention.

**Static vs contextual embeddings:** one vector per word (Word2Vec) vs per occurrence (BERT) — the "bank" test.

**Stemming vs lemmatization:** chop vs dictionary. **TF-IDF vs embeddings:** sparse counts vs dense meaning.

**GAN vs diffusion vs VAE:** adversarial / iterative denoising (SOTA) / latent-probabilistic.

**CNN vs dense on images:** local + shared weights + hierarchy vs parameter explosion. **CNN vs ViT:** spatial priors vs patch-tokens + scale.

**Classification vs detection vs segmentation:** image label / boxes+labels / per-pixel.

**Precision vs recall:** false-positive cost vs false-negative cost; F1 harmonizes.

## One-Formula-Each
Attention = softmax(QKᵀ/√d_k)V · cosine sim = a·b/(|a||b|) · tf-idf = tf × log(N/df) · conv output = (N−K+2P)/S + 1.

## Ties to Your Other Artifacts
Vector search/HNSW + inverted index → `System_Design/Building_Blocks/15_SEARCH_INVERTED_INDEX.md` · serving LLM apps (caching, queues, rate limits) → System Design building blocks · GPU training pipelines are batch systems → Job Scheduler case study.
