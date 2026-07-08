# CDN (Content Delivery Network)

## 1. What Problem It Solves

Physics: a user in India fetching an image from a server in Virginia pays ~200ms+ round trip, every time, and your origin server pays bandwidth for every user. A CDN caches content on servers ("edge nodes / PoPs") physically near users — cutting latency to ~10–50ms and offloading most traffic from your origin.

## 2. How It Works

- User requests `img.example.com/cat.jpg` → DNS resolves to the *nearest* edge node (GeoDNS or anycast).
- Edge has it cached → serve immediately (**cache hit**). Doesn't → fetch from **origin** (your servers or object storage like S3), cache it, serve it (**cache miss**).
- Freshness controlled via HTTP headers: `Cache-Control: max-age`, `ETag`/`If-None-Match` revalidation. Explicit **purge/invalidation** APIs for immediate updates; more common trick: **versioned URLs** (`app.v2.js`, content-hash filenames) — never invalidate, just reference a new URL.

**Push vs Pull:**

| | Pull (default) | Push |
|---|---|---|
| How | Edge fetches from origin on first miss | You upload content to CDN ahead of time |
| Good for | Long tail, most web assets | Large files with predictable demand (video releases) |
| Cost | First user per region eats miss latency | Storage cost for possibly-unwatched content |

**What CDNs serve:** static assets (images, JS/CSS, video segments) primarily; modern CDNs also do dynamic content acceleration (smart routing over their backbone), edge compute (Cloudflare Workers, Lambda@Edge), TLS termination, and DDoS absorption.

**Video streaming:** video is chunked (HLS/DASH segments of 2–10s at multiple bitrates); the CDN caches segments — this is >90% of YouTube/Netflix egress.

## 3. Tradeoffs

**Pros:** big latency win globally, origin offload (often 90%+ of bandwidth), DDoS buffer, availability (origin down → edges can serve cached content).

**Cons:** cost per GB, invalidation/staleness complexity, cache hit rates suffer for long-tail or personalized content, edge behavior can be harder to debug.

**When NOT to use:** highly personalized or rapidly-changing responses (per-user API data — cache hit rate ≈ 0), internal apps with users in one region near servers, strict data-locality/compliance constraints on where bytes may be cached.

## 4. Diagram

```
        ┌────────── Edge PoP (Mumbai) ◀──── users in India (~20ms)
        │                 │ miss
Origin ─┤◀── cache miss ──┘
(S3/servers)
        │                 ┌ hit: served locally
        └────────── Edge PoP (Frankfurt) ◀── users in EU (~15ms)

Without CDN: every user → origin (100–300ms + origin bandwidth $$$)
With CDN:    ~95% of requests never reach origin
```

## 5. Common Interview Follow-ups

**Q: How do users get routed to the nearest edge?**
A: GeoDNS (DNS answers differ by resolver location) or anycast (same IP announced from many locations; BGP routes to nearest). Anycast reacts faster to failures.

**Q: How do you invalidate CDN content instantly?**
A: Purge API (seconds to propagate) — but the better design is immutable, content-hashed URLs so "invalidation" is just deploying HTML that references new asset names.

**Q: What's a good CDN strategy for user-uploaded images?**
A: Store originals in object storage (S3), serve via CDN with long max-age and content-hash URLs; generate resized variants on upload (or on-the-fly with edge resizing + cache).

**Q: CDN vs your own cache (Redis)?**
A: CDN caches HTTP responses geographically close to *users*, mostly static. Redis caches data close to *your app*, for dynamic queries. Complementary layers, not alternatives.

**Q: What if the origin goes down?**
A: Configure edges to serve stale content on origin error (`stale-if-error`) — degraded freshness beats an outage.

---
**Used in case studies:** YouTube (dominant component), News Feed (media), Pastebin, Chat (media attachments).
