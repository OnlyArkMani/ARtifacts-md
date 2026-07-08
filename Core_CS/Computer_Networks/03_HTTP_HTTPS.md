# HTTP & HTTPS

> **Tag: Theory/recall** — methods, status codes, HTTPS handshake, and HTTP version differences; heavily asked for web/backend roles.

## Concept

HTTP: stateless request-response protocol over TCP (or QUIC). Client sends method + path + headers (+ body); server replies status + headers (+ body). **Stateless** = each request independent → sessions built on top via cookies/tokens.

## Methods

| Method | Use | Idempotent? | Safe? |
|---|---|---|---|
| GET | Fetch; no body semantics | ✅ | ✅ |
| POST | Create / process | ❌ | ❌ |
| PUT | Replace entire resource | ✅ | ❌ |
| PATCH | Partial update | ❌ (by spec) | ❌ |
| DELETE | Remove | ✅ | ❌ |
| HEAD / OPTIONS | Headers only / capabilities (CORS preflight) | ✅ | ✅ |

Idempotent = repeating has same effect (safe to retry — links to System Design idempotency). PUT vs POST and GET vs POST are guaranteed questions.

## Status Codes (know these ~15)

- **2xx:** 200 OK · 201 Created · 204 No Content
- **3xx:** 301 permanent redirect · 302 temporary · 304 Not Modified (cache validation!)
- **4xx (client's fault):** 400 Bad Request · **401 Unauthenticated** · **403 Unauthorized/forbidden** · 404 · 405 Method Not Allowed · 409 Conflict · **429 Too Many Requests** (rate limiting)
- **5xx (server's fault):** 500 Internal Error · 502 Bad Gateway (upstream died) · 503 Unavailable (overload) · **504 Gateway Timeout** (upstream slow)

401 vs 403 ("who are you?" vs "you can't do that") and 502 vs 504 are the favorite pairs.

## HTTP Versions

| | 1.0 | 1.1 | 2 | 3 |
|---|---|---|---|---|
| Connection | New TCP per request | **Keep-alive** persistent, pipelining (broken) | One TCP, **multiplexed streams**, header compression (HPACK), server push | **QUIC over UDP**, no TCP HoL blocking, 0–1 RTT setup |
| Head-of-line problem | — | At HTTP level (serial requests) | Solved at HTTP level, remains at TCP level | Solved fully (independent streams) |

Story to tell: each version attacks the previous one's head-of-line blocking one level deeper.

## HTTPS = HTTP over TLS

Provides **confidentiality** (encryption), **integrity** (tamper detection), **authentication** (certificates).

**TLS 1.3 handshake (simplified):**
1. ClientHello: supported ciphers + client key share
2. ServerHello: chosen cipher + server key share + **certificate**
3. Client verifies certificate against trusted CAs (chain of trust), both derive session keys (ECDHE) → encrypted from here. 1 RTT (TLS1.2 took 2).

- **Asymmetric crypto** (slow) authenticates & agrees keys; **symmetric** (fast, AES) encrypts the session — the hybrid design is a favorite question.
- **Certificate:** server's public key + identity, signed by a CA the browser trusts (chain: leaf ← intermediate ← root).
- **Forward secrecy:** ephemeral keys per session (ECDHE) — stealing the server's long-term key later can't decrypt recorded traffic.

## Cookies, Sessions, Tokens (statefulness on top)

Cookie = server-set key-value auto-sent on future requests. Session: server-side state, cookie holds session ID. JWT: signed self-contained claims, no server state (scales; but revocation is hard). Flags: `HttpOnly` (no JS access — XSS defense), `Secure`, `SameSite` (CSRF defense).

## Most-Asked Interview Questions

1. **GET vs POST?** Semantics (fetch vs create), idempotency, caching, URL vs body params, bookmark/history behavior — not "GET is insecure" (both plaintext without TLS; GET params leak in logs though).
2. **PUT vs POST vs PATCH?** PUT replaces at a known URI (idempotent); POST creates under a collection (server picks URI); PATCH partial.
3. **401 vs 403? 502 vs 504?** → above.
4. **How does HTTPS work / TLS handshake?** → 3 steps + hybrid crypto reasoning + cert chain.
5. **HTTP/1.1 vs 2 vs 3?** → head-of-line blocking narrative.
6. **How do sessions work if HTTP is stateless?** Cookies → session store or JWT; tradeoffs (server memory vs revocation).
7. **What is caching in HTTP?** `Cache-Control: max-age`, ETag + `If-None-Match` → 304. Ties directly to CDN behavior (cross-ref System Design CDN).
8. **What is CORS?** Browser same-origin policy relaxation: server opts in via `Access-Control-Allow-*`; preflight OPTIONS for non-simple requests. It protects *users' browsers*, not your API.
