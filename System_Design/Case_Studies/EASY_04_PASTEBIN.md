# Case Study: Pastebin (Easy)

**Building blocks used:** SQL vs NoSQL · Object Storage + CDN · Caching · Replication · Rate Limiting

Very similar skeleton to URL Shortener — the twist is **storing content (KBs–MBs), not just a mapping**, which introduces the metadata-vs-blob split. If you've done URL Shortener, focus on the differences.

---

## 1. Basic Version (single server)

API:
- `POST /pastes {content, ttl?, visibility?}` → `{paste_id}` (Base62, as in URL Shortener)
- `GET /{paste_id}` → content (render or raw)

Single table: `(paste_id PK, content TEXT, created_at, expires_at, visibility)`. Fine until content size × volume outgrows the DB.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Create paste (up to ~10 MB), read via link; optional: expiry, private/unlisted, syntax highlighting, edit/delete by owner.

### Non-functional
Read-heavy (~10:1); read latency <200ms; high availability; pastes are immutable after creation (huge simplification — cache freely); durability matters (people paste important things).

### Capacity estimate
- 10M new pastes/day → ~100 writes/s. Reads ~1k/s, peak 5k/s. Modest.
- Storage is the story: avg 10 KB × 10M/day = **100 GB/day → ~36 TB/yr** — too much for a relational DB's comfort, trivial for object storage. ⇒ **split metadata from content**.

### High-level design

```
 Client ──▶ LB ──▶ App servers ──▶ Redis (hot paste metadata+small bodies)
                     │    │ miss
        write        │    ▼
   ┌─────────────────┘  ┌────────────┐  metadata: id, owner, expiry,
   ▼                    │ DB         │  visibility, s3_key
┌─────────────┐         └────────────┘
│ Object store│ ◀── content blobs (S3)
│  + CDN      │ ──▶ reads of popular pastes served from edge
└─────────────┘
```

- **Metadata** (small, queryable) in a replicated DB; **content** in object storage (11-nines durability, effectively infinite, cheap).
- Public popular pastes served **via CDN** — immutability makes cache headers trivial (`max-age=∞` keyed by ID).
- Small pastes (<~10 KB) can be inlined in the DB/cache to skip an S3 round trip — nice optimization to mention.

### Deep dive 1: expiry (TTL) at scale
Don't scan-and-delete synchronously. Lazy check on read (expired → 404) + background sweeper deleting expired metadata and S3 objects in batches. If using DynamoDB: native TTL. S3: lifecycle rules per prefix. Point: **expiry is an eventual-cleanup problem, correctness enforced at read time.**

### Deep dive 2: read path & caching
Cache-aside Redis for metadata + hot small bodies; CDN for public raw content. Because pastes are immutable, there is **no invalidation problem** — only deletion (handle with a tombstone check or short CDN TTL for deletable pastes; or make delete = new URL policy). Private pastes bypass CDN (auth check in app) — or use signed URLs with short expiry.

### Bottlenecks & fixes
- Viral paste (hot key) → CDN absorbs it; that's the design's whole point.
- Abuse (malware hosting, doxxing) → rate limit writes, content scanning pipeline (async), report/takedown flow.
- DB growth → metadata is small; shard by paste_id hash if ever needed.
- Large uploads → direct-to-S3 presigned upload URLs, app never proxies bytes.

### Follow-up questions
- **Why not store content in the DB?** Cost, backup bloat, replication amplification, DBs are bad at multi-MB blobs; object storage is purpose-built.
- **How do private pastes work with a CDN?** Don't cache them publicly; serve via app with auth, or short-lived signed CDN URLs.
- **Edit support?** Either versioned objects (new S3 key per version, metadata points to latest) or forbid edits (many real pastebins do).
- **How is this different from URL shortener?** Same ID/read-heavy skeleton + blob storage layer + expiry + abuse surface. Good to say explicitly — shows pattern recognition.
- **Search over pastes?** Async index public pastes into Elasticsearch (see `../Building_Blocks/15_SEARCH_INVERTED_INDEX.md`); never search S3 directly.

### Tradeoffs to defend
Object storage + metadata DB (two systems, right tools) vs one DB (simpler, doesn't scale storage); CDN caching of public content (fast, cheap) vs deletability (stale window); inline small pastes (latency) vs uniform path (simplicity).

## 3. Advanced Version

- **Multi-region:** S3 cross-region replication + regional read caches; metadata DB with regional replicas (immutable data = easy). Writes can home to one region — write latency is tolerable.
- **Failure handling:** S3 outage in a region → serve from replica region (stale-if-error at CDN helps); DB down → cached reads still work.
- **Compliance:** takedown pipeline, hash-matching known-bad content at upload, retention policies per jurisdiction.
