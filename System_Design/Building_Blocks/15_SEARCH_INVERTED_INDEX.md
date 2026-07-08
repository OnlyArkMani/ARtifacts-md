# Search & Inverted Indexes

## 1. What Problem It Solves

`WHERE text LIKE '%pizza%'` scans every row and can't rank results — useless at scale. Search needs the reverse mapping: not "document → words" but **"word → documents containing it"**. That's an inverted index, the core of Elasticsearch, Lucene, and every search engine.

## 2. How It Works

### Building the index

1. **Analyze** each document: tokenize ("Best Pizza in NYC!" → `best, pizza, in, nyc`), lowercase, drop stopwords, **stem/lemmatize** (`running` → `run`).
2. For each term, store a **postings list**: sorted doc IDs containing it (plus term positions for phrase queries, and term frequency for ranking).

```
"pizza"  → [doc2, doc5, doc9, doc12]
"nyc"    → [doc5, doc7, doc12]
```

### Querying

- `pizza AND nyc` → **intersect** postings lists (fast on sorted lists) → [doc5, doc12].
- Phrase "new york" → intersect + check adjacent positions.
- Prefix/fuzzy → term dictionary structures (tries/FSTs), edit-distance automata.

### Ranking

- **TF-IDF / BM25:** score docs higher when the term is frequent in the doc but rare across the corpus. BM25 is the modern default.
- Real ranking blends text score with signals: recency, popularity, personalization.
- **Semantic/vector search:** embed docs and queries as vectors, retrieve nearest neighbors (ANN indexes like HNSW) — catches synonyms BM25 misses. Modern systems do **hybrid** (BM25 + vector, merged).

### Scaling (Elasticsearch model)

- Index split into **shards** by hashing doc ID; each shard is a full mini-index, replicated.
- Query fans out to all shards (**scatter**), each returns its top-k, coordinator merges (**gather**).
- **Near-real-time:** writes buffer and become searchable after a refresh interval (~1s) — search indexes are eventually consistent by design.
- Index is populated *from* the source-of-truth DB via CDC/queues — search is a derived, rebuildable view, never the primary store.

## 3. Tradeoffs

**Pros:** millisecond full-text search over billions of docs; relevance ranking; faceting/aggregations; typo and language tolerance.

**Cons:** another cluster to run; eventual consistency with the source DB (dual-write drift — use CDC); index storage can rival the data itself; deep pagination and huge scatter-gathers are expensive.

**When NOT to use:** exact-match lookups (a DB index is simpler); small datasets (Postgres full-text may suffice — say this, it shows judgment); as a system of record (it's a derived index — you must be able to rebuild it).

## 4. Diagram

```
 Write path:                       Query path: "pizza nyc"
 DB (source of truth)                     │ analyze
   │ CDC / queue                          ▼
   ▼                            ┌── shard1: top-k ──┐
 Indexer → analyze → postings   │── shard2: top-k ──│→ coordinator
   ▼                            └── shard3: top-k ──┘   merge+rank
 Inverted index shards                                  → top 10
 "pizza"→[2,5,9]  "nyc"→[5,7]        (scatter-gather)
```

## 5. Common Interview Follow-ups

**Q: Why is LIKE '%term%' slow but the inverted index fast?**
A: Leading wildcard defeats B-tree ordering → full scan. Inverted index precomputes term→docs, so lookup is a hash/tree hit plus list intersection.

**Q: How do you keep the search index in sync with the database?**
A: Not dual writes from the app (drift on partial failure). Stream changes: CDC (Debezium) or outbox pattern → queue → indexer. Accept ~seconds of lag; rebuild from source when needed.

**Q: How does Elasticsearch handle a search across shards?**
A: Scatter-gather: query all shards, each scores locally and returns top-k IDs, coordinator merges and fetches. Costs grow with shard count and page depth (why deep pagination is discouraged — use search_after).

**Q: TF-IDF in one sentence?**
A: Score = term frequency in this doc × rarity of term across all docs — "pizza" appearing 5 times matters; "the" appearing 50 times doesn't.

**Q: When would you add vector search?**
A: When lexical match misses intent — synonyms, natural-language queries, recommendations. Hybrid retrieval (BM25 + embeddings via HNSW) then rerank is the standard modern stack.

---
**Used in case studies:** Web Crawler (feeds a search index), YouTube (video search), News Feed (content discovery). Cross-ref: `11_INDEXING.md`.
