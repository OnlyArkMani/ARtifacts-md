# NLP — Interview Prep

> **Tag: Theory + pipeline knowledge** — expect the classic pipeline (preprocess → represent → model), the evolution story to Transformers, and standard tasks/metrics.

## The Text Representation Evolution (the narrative to know)

1. **Bag of Words / One-hot:** count words per document. Sparse, huge, no meaning ("good" ⊥ "great"), no order.
2. **TF-IDF:** weight = term frequency × inverse document frequency — frequent-here but rare-overall words matter most. Still sparse/orderless, but a strong baseline for search & classification (same TF-IDF as the inverted-index file in System Design).
3. **Word embeddings (Word2Vec/GloVe, 2013+):** dense ~300-d vectors from co-occurrence — **similar words get nearby vectors**; famous arithmetic: king − man + woman ≈ queen. Limitation: one static vector per word — "bank" (river/money) collapsed.
4. **Contextual embeddings (ELMo/BERT, 2018+):** the vector for "bank" depends on its sentence — computed by a deep model over the whole context. This + pretraining/fine-tuning = the modern era.
5. **LLMs:** one giant pretrained model, prompted or lightly tuned, replaces most task-specific pipelines.

Word2Vec detail worth knowing: train a small net to predict context from word (skip-gram) or word from context (CBOW); the *weights* become the embeddings — the prediction task is scaffolding.

## Classic Preprocessing (still asked)

**Tokenization** (words → units; modern: subword/BPE — handles rare words, fixed vocab) · lowercasing · **stopword removal** (the/is/at — for classical models; NOT for Transformers, they need them) · **stemming** (chop suffixes: "running"→"runn", crude) vs **lemmatization** (dictionary-informed: "better"→"good", clean) · n-grams (capture local phrases).

## Standard Tasks & How They're Done Now

| Task | What | Classic → Modern |
|---|---|---|
| Text classification | spam, sentiment, topics | TF-IDF + Naive Bayes/LogReg → fine-tuned BERT / LLM prompt |
| **NER** | find entities (people, orgs, dates) | CRFs → token-classification BERT |
| Machine translation | | phrase tables → seq2seq+attention → Transformers (its birthplace) |
| Summarization | extractive (pick sentences) vs abstractive (write new) | → encoder-decoder (T5/BART), LLMs |
| Question answering | span extraction / generative | → BERT-span → RAG + LLMs |
| Sentiment | polarity ± aspect-level | lexicons → fine-tuned models |
| Semantic similarity/search | meaning-match | TF-IDF cosine → **sentence embeddings + vector search** (RAG's retrieval half) |

**Similarity math:** cosine similarity = dot(a,b)/(|a||b|) — angle, not magnitude; the standard for embeddings.

**Metrics:** classification → accuracy/precision/recall/**F1** (know the precision-recall tradeoff story: spam filter vs cancer screen); translation → BLEU; summarization → ROUGE; LMs → perplexity (lower = better next-token prediction); modern LLM outputs → human/LLM-as-judge eval (say that metrics are an open problem).

## Evergreen Concepts

- **Why is NLP hard?** Ambiguity at every level: lexical ("bank"), syntactic ("I saw the man with the telescope"), coreference ("it"), sarcasm/pragmatics, world knowledge.
- **Sequence labeling vs sequence classification vs seq2seq** — one label per token / per text / text→text: places every task.
- **Pretrain → fine-tune paradigm:** general language knowledge once, task adaptation cheaply — the transfer-learning story (same as CV's ImageNet story, one file over).

## Most-Asked Interview Questions

1. **TF-IDF — explain and compute.** tf × log(N/df); score a toy example; still the search baseline.
2. **Word2Vec vs TF-IDF?** Dense semantic vectors from prediction task vs sparse count statistics; similarity works vs doesn't.
3. **Static vs contextual embeddings?** One vector per word vs per-occurrence; the "bank" example.
4. **Stemming vs lemmatization?** Crude suffix-chopping vs vocabulary-aware normalization.
5. **How does BERT differ from GPT for NLP tasks?** Bidirectional understanding (classification/NER) vs generative (see `02_TRANSFORMERS.md` table).
6. **Precision vs recall — which matters when?** False-positive-costly (spam→inbox) vs false-negative-costly (disease screening); F1 balances.
7. **How would you build a sentiment classifier today?** Honest answer has tiers: LLM prompt (fastest) → fine-tuned small transformer (volume/cost) → TF-IDF+LogReg (baseline/interpretable); choosing by constraints impresses.
8. **What is BPE/subword tokenization and why?** Open vocabulary from finite units; rare words decompose instead of becoming UNK.
9. **How does semantic search differ from keyword search?** Embedding-space nearest neighbors vs term matching; hybrid wins (cross-ref System Design search file).
