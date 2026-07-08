# Transformers — Interview Prep

> **Tag: Theory** — "explain attention" is the core question. Build the answer in layers: intuition → mechanism → architecture → variants.

## Why Transformers Exist

Before 2017: RNNs/LSTMs processed text sequentially — slow (no parallelism), and long-range dependencies faded across steps. **"Attention Is All You Need" (2017)** dropped recurrence entirely: every token looks at every other token *directly* (constant path length), and everything computes in parallel → scales beautifully with data + GPUs. That scaling is what made LLMs possible.

## Self-Attention (the answer to rehearse)

**Intuition:** in "The animal didn't cross the street because *it* was tired," resolving "it" requires looking back at "animal." Self-attention lets every token compute *how relevant every other token is to it* and blend their information accordingly.

**Mechanism:** each token's embedding is projected into three vectors — **Query** (what I'm looking for), **Key** (what I contain/advertise), **Value** (what I give if attended to):

```
Attention(Q, K, V) = softmax(QKᵀ / √d_k) · V
```

1. Q·Kᵀ — every token's query dotted with every token's key = relevance scores (n×n matrix).
2. ÷√d_k — scale so softmax doesn't saturate (know *why* it's there).
3. softmax — scores → weights summing to 1.
4. Weighted sum of Values — each token's new representation = blend of what it attended to.

**Multi-head attention:** run h attention operations in parallel with separate projections — each head can specialize (syntax, coreference, position patterns) — then concatenate. Richer than one attention "view."

**Positional encoding:** attention is order-blind (a set operation) → inject position info into embeddings (sinusoidal or learned; modern: RoPE) so "dog bites man" ≠ "man bites dog."

## The Block & The Architecture

```
Input tokens → embeddings + positions
   ┌────────────────────────────┐
   │  Multi-head self-attention │ ← residual + layer norm
   │  Feed-forward network      │ ← residual + layer norm
   └────────────────────────────┘  × N layers (dozens–100+)
→ output distribution over vocabulary
```

Residual connections + layer norm make deep stacks trainable; the FFN (per-token MLP) is where much of the "knowledge" capacity lives.

**Three architecture families (classic question):**

| | Encoder-only | Decoder-only | Encoder-decoder |
|---|---|---|---|
| Attention | Bidirectional (sees both sides) | **Causal/masked** (only left context) | Both + cross-attention |
| Pretraining | Masked-token prediction | Next-token prediction | Denoising/seq2seq |
| Good at | Understanding: classification, embeddings, NER | **Generation** — all modern LLMs | Translation, summarization |
| Examples | BERT | **GPT, Llama, Claude** | T5, original Transformer |

**Causal masking:** during training the decoder predicts every position simultaneously but can only attend leftward (future positions masked to −∞ before softmax) — enables parallel training of a generative model.

## Costs & Variants (depth signals)

- Self-attention is **O(n²)** in sequence length — the context-window bottleneck. Mitigations: sparse/sliding-window attention, FlashAttention (exact but memory-efficient), grouped-query attention (**KV-cache** shrinking — also know: at inference, past tokens' K/V are cached so each new token costs one step, not a recompute).
- Scaling laws: loss falls predictably with model + data + compute — why "bigger" kept winning.
- Fine-tuning efficiently: **LoRA** — train tiny low-rank adapter matrices instead of all weights (name it, one line).

## Most-Asked Interview Questions

1. **Explain self-attention / Q, K, V.** → intuition sentence + 4-step mechanism + the formula.
2. **Why did Transformers replace RNNs?** Parallel training + direct long-range connections vs sequential bottleneck + fading gradients.
3. **Why multi-head?** Multiple relevance patterns in parallel (heads specialize).
4. **Why positional encodings?** Attention is permutation-invariant; order must be injected.
5. **BERT vs GPT?** Bidirectional encoder for understanding vs causal decoder for generation; masked-LM vs next-token objectives.
6. **Why √d_k scaling? Why residuals/layer norm?** Softmax saturation; gradient flow in deep stacks.
7. **Why is long context expensive?** O(n²) attention + KV-cache memory growth.
8. **What happens at inference step-by-step?** Tokenize → forward pass with cached K/V → logits → sample → append → repeat (connects to GenAI file).
