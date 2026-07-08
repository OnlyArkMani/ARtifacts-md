# Case Study: YouTube-Scale Video Platform (Hard)

**Building blocks used:** CDN (dominant) · Object Storage · Message Queues (transcoding pipeline) · Caching · Data Partitioning · Search · SQL vs NoSQL

Two nearly-independent systems: the **upload/processing pipeline** (async, throughput) and the **serving path** (CDN, latency). Recognizing that split *is* the answer's skeleton.

---

## 1. Basic Version (single server)

Upload MP4 → save to disk → serve file via HTTP range requests (`<video>` handles seeking). Metadata (title, views) in one DB table. Works for a class project; dies on: one bitrate for all networks, one server's bandwidth, no processing.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Upload videos; transcode to multiple qualities; stream with adaptive bitrate; metadata (titles, thumbnails); view counts; search (light). (Cut: recommendations, comments, live streaming — flag as extensions.)

### Non-functional
- Scale: 500 hrs uploaded/min; 5M concurrent viewers
- Startup latency <1–2s; smooth playback everywhere (variable networks)
- Durability: never lose a video; Availability: watching >> uploading
- Consistency: view counts eventual (nobody can verify); metadata eventual is fine

### Capacity estimate
- Egress: 5M viewers × 3 Mbps = **15 Tbps** → *forces* CDN; origin can't do this. The single most important number in this design.
- Upload: 500 hr/min × ~3 GB/hr raw ≈ 25 GB/s ingest → parallel chunked uploads to object storage.
- Storage: with multi-bitrate copies ~2× raw → **PB/month scale** → tiered object storage.

### High-level design

```
UPLOAD (async pipeline)                 SERVING (hot path)
Creator                                  Viewer
  │ chunked upload (presigned)             │ GET manifest
  ▼                                        ▼
Object storage (raw)                    ┌─────────┐
  │ event → Kafka                       │   CDN   │ 95%+ hit
  ▼                                     └────┬────┘
┌──────────────────┐                         │ miss
│ Transcoding fleet │ per-chunk parallel     ▼
│ → H.264/AV1 ×     │ (DAG: audio, video  Origin (object storage,
│ 240p…4K → HLS/DASH│  per-res, thumbs,    segments + manifests)
│ segments (2–10s)  │  captions)
└──────┬───────────┘                    Metadata svc ── DB (sharded)
       ▼                                     + Redis cache
Object storage (processed) ──▶ CDN pre-warm popular
Metadata DB: status uploading→processing→live
```

### Deep dive 1: transcoding pipeline
- Video split into chunks → **transcode chunks in parallel** across workers (a 2-hour video finishes in ~minutes) → stitch manifests. Model as a DAG (inspect → per-resolution encode → thumbnails → captions → assemble) on a queue-driven worker fleet.
- At-least-once + **idempotent chunk tasks** (re-encode of same chunk = same output, keyed by content hash) — crash-safe by construction.
- Priority queues: new upload from big creator > re-encode backfills. Spot/preemptible instances for cost — flag it.

### Deep dive 2: adaptive bitrate streaming (why segments matter)
- Each quality → **2–10s segments** + a manifest (HLS/DASH). Player measures its own bandwidth and picks the next segment's quality — mid-playback switching, seek-friendly, and **each segment is an independently CDN-cacheable HTTP object** — the whole serving architecture falls out of this format choice.
- Popular content: pre-pushed to edges; long tail: pull-through on miss. Origin shielding (regional mid-tier caches) so a miss doesn't multiply against object storage.

### Deep dive 3 (if pushed): view counting at scale
Millions of increments/s on hot videos → don't UPDATE a row per view. Client events → Kafka → streaming aggregation (per-video counts in windows) → periodic flush to DB; serve counts from cache; exactness unnecessary (say so) — dedupe bots downstream. Classic write-heavy → stream-aggregate pattern, reusable in many designs.

### Bottlenecks & fixes
- Viral video ("hot key" at CDN): it's what CDNs are for; ensure origin shield + pre-warm on trending signals.
- Upload spikes: queue absorbs; transcoding autoscales; publish latency degrades gracefully (priority lanes protect key creators).
- Metadata hot rows (viral video's title/stats): cache with short TTL + stream-updated counters.
- Storage cost: cold videos → infrequent-access/archival tiers; delete unused transcodes, re-generate on demand (compute-storage tradeoff — nice flourish).

### Follow-up questions
- **Why HLS segments and not one big file?** Cacheability, seeking, mid-stream quality switching, resumable delivery, parallel fetch.
- **How does resume-upload work?** Chunked upload with per-chunk ETags; client retries missing chunks (idempotent PUTs) — same pattern as multipart S3.
- **Copyright detection?** Fingerprint (perceptual hash) pipeline stage against a reference DB — async, flag before publish. 
- **Live streaming difference?** Segments produced in real time, tiny buffer, no full pre-transcode; latency-vs-quality knob (LL-HLS); fan-out identical (CDN).
- **Why is watching available even if uploads break?** The two pipelines share almost nothing — serving reads immutable objects via CDN. Deliberate isolation; say it.

### Tradeoffs to defend
CDN cost vs origin egress (no real choice at 15 Tbps); pre-transcode all qualities (storage) vs on-demand (compute, latency); eventual view counts (scale) vs exact (pointless); AV1/H.265 (bandwidth −30%) vs encode cost & device support.

## 3. Advanced Version

- **Multi-region:** uploads ingest to nearest region → replicate processed output to global object storage + CDN; metadata DB regional replicas (eventual); user "home" region for creator data.
- **Failure:** region loss → CDN reroutes to surviving origins (content replicated); transcoding backlog drains cross-region; publish delayed, playback unaffected — availability asymmetry honored.
- **Recommendations:** watch events → Kafka → feature pipelines → candidate generation + ranking (two-stage, like News Feed advanced) — separate system, mention only.
- **Cost engineering:** per-title encoding (bitrate tuned per content complexity), storage tiering by popularity decay, codec migration waves — at this scale, cost IS an architectural requirement.
