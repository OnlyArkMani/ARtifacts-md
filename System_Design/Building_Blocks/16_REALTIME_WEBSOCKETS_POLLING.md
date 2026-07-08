# Real-Time Delivery: WebSockets, Long Polling, SSE

## 1. What Problem It Solves

HTTP is request-response: the server can't speak unless spoken to. But chat messages, live scores, driver locations, and notifications originate at the *server*. You need a way to push data to clients the moment it happens — without clients hammering the server with "anything new?" requests.

## 2. How It Works

| Technique | Mechanism | Latency | Cost | Direction |
|---|---|---|---|---|
| **Short polling** | Client asks every N seconds | up to N sec | Wasteful (mostly empty responses) | client-pull |
| **Long polling** | Server holds the request open until data arrives (or timeout), client immediately re-requests | Near-instant | Connection churn, per-request overhead | client-pull that feels like push |
| **SSE** (Server-Sent Events) | One long-lived HTTP response streaming events | Instant | Cheap; auto-reconnect built in | **server→client only** |
| **WebSocket** | HTTP Upgrade handshake → persistent full-duplex TCP channel | Instant | Persistent connection state per client | **bidirectional** |

**Choosing:** need client→server AND server→client at low latency (chat typing, games, collaborative editing) → WebSocket. Only server→client (notifications, feeds, dashboards) → SSE is simpler and proxy-friendly. Legacy/compat fallback → long polling.

### Scaling WebSockets (the real interview meat)

- Each server holds ~100k–1M concurrent connections (memory + FD limits). Millions of users → a fleet of **connection/gateway servers**.
- **The routing problem:** user A (on server 1) messages user B (on server 7). Server 1 must find B's server. Solutions:
  - **Service registry:** Redis map `user_id → gateway_id` updated on connect/disconnect; sender's server looks up and forwards.
  - **Pub/sub backbone:** every gateway subscribes to channels for its connected users (Redis pub/sub, Kafka); message published to `user:B` channel reaches whichever gateway holds B.
- **Stateful LB concern:** connections are sticky by nature (L4 LB / connection-aware routing); a gateway dying drops its connections → clients auto-reconnect (with jittered backoff to avoid a reconnect stampede — "thundering herd").
- **Heartbeats** (ping/pong) detect dead connections and keep NATs/proxies from closing idle ones.
- Offline users: push through mobile push services (APNs/FCM) instead; queue messages for delivery on reconnect.

## 3. Tradeoffs

**Pros (WebSocket):** true real-time, bidirectional, low per-message overhead after handshake.

**Cons:** server must hold per-connection state (memory, harder autoscaling/deploys — draining connections), trickier through proxies/corporate firewalls, reconnect/resume logic is on you (missed-message recovery needs sequence numbers or a fetch-since API).

**When NOT to use:** data that changes rarely (poll every minute is fine and vastly simpler); one-way streams (SSE simpler); request-response APIs (plain HTTP). Don't reach for WebSockets just because "real-time" sounds good.

## 4. Diagram

```
        WS       ┌──────────┐     pub/sub      ┌──────────┐    WS
User A ◀────────▶│ Gateway 1│◀───────────────▶│ Gateway 7 │◀────────▶ User B
                 └────┬─────┘  publish to      └──────────┘
                      │        channel user:B
                      ▼
              ┌───────────────┐   B offline? ──▶ push (FCM/APNs)
              │ Chat service  │──▶ persist msg   + queue for reconnect
              └───────────────┘
Registry: { A→gw1, B→gw7 }  (Redis, updated on connect/disconnect)
```

## 5. Common Interview Follow-ups

**Q: Long polling vs WebSocket — when is long polling actually fine?**
A: Moderate-frequency updates, simple infra, environments that break WebSockets. It approximates push with plain HTTP. At high message rates its per-request overhead loses badly.

**Q: How do you deliver a message to a user connected to a different server?**
A: Registry lookup (user→gateway) + direct forward, or pub/sub channels per user/conversation. Both patterns are standard; pub/sub scales fan-out better, registry is more targeted.

**Q: A gateway with 500k connections dies. What happens?**
A: All 500k clients reconnect at once → stampede. Mitigate: jittered exponential backoff on clients, connection draining on planned deploys, spare capacity headroom, and message persistence so nothing is lost — clients fetch missed messages by sequence number on reconnect.

**Q: How do you know a client is still connected?**
A: Heartbeat ping/pong at intervals; missed N pongs → close and clean up registry. TCP alone won't tell you promptly.

**Q: How would you send notifications to 10M offline mobile users?**
A: Not WebSockets — APNs/FCM push via a notification service with queues, batching, rate limiting, and dedup. WebSockets are for *active* sessions.

---
**Used in case studies:** Chat Application (core), Ride-Sharing (location streams), Notification System, News Feed (live updates).
