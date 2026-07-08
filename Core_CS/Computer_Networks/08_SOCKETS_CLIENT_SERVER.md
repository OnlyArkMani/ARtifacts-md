# Sockets & Client-Server Communication

> **Tag: Applied** — the socket API call sequence is asked directly, sometimes as "write pseudo-code for a server."

## Concept

A **socket** is the OS's endpoint abstraction for network communication — to the app it's a file descriptor you read/write; underneath, the kernel handles TCP/UDP/IP. Identified by the 5-tuple: (protocol, src IP, src port, dst IP, dst port) — this tuple uniqueness is how one server port serves thousands of clients simultaneously.

## The Call Sequence (memorize both sides)

```
TCP SERVER                       TCP CLIENT
socket()   create fd             socket()
bind()     attach to IP:port
listen()   queue for connections
accept()   block → NEW fd per client   connect()  ── 3-way handshake ──▶ (accept returns)
read()/write()  ◀───── data exchange ─────▶  write()/read()
close()                          close()
```

Key subtlety interviewers probe: **`accept()` returns a NEW socket** per client; the listening socket keeps listening. The server socket is (proto, srcIP, 80, *, *) while each connection socket is fully qualified by the client's IP:port — no port conflict.

UDP: no listen/accept/connect — just `socket() → bind() → recvfrom()/sendto()` (each datagram carries addressing).

## Minimal Python (shows understanding without boilerplate)

```python
# Server                                  # Client
import socket                             import socket
s = socket.socket()                       c = socket.socket()
s.bind(("0.0.0.0", 8080))                 c.connect(("host", 8080))
s.listen(5)          # backlog            c.send(b"hello")
while True:                               print(c.recv(1024))
    conn, addr = s.accept()   # new fd!   c.close()
    data = conn.recv(1024)
    conn.send(data.upper())
    conn.close()
```

## Handling Many Clients (scaling story — bridges to OS & System Design)

1. **Thread/process per connection:** simple; ~10k+ threads = memory + context-switch pain (the **C10K problem**).
2. **I/O multiplexing:** one thread watches many sockets — `select` (O(n) scan, fd limits) → `poll` → **`epoll`/kqueue** (O(1) event notification) — how Nginx/Redis/Node serve 100k+ connections.
3. **Event loop + non-blocking sockets:** epoll + callbacks/coroutines (asyncio, Node) — concurrency without threads (cross-ref OS `07_CONCURRENCY_MULTITHREADING.md`, System Design WebSockets).

**Blocking vs non-blocking:** blocking `read()` sleeps until data; non-blocking returns EWOULDBLOCK immediately — the building block of event loops.

## Most-Asked Interview Questions

1. **Walk through the socket calls for a TCP server.** → sequence above; emphasize accept()'s new fd.
2. **How can one port handle thousands of clients?** Connections distinguished by full 5-tuple; the port is shared, the (clientIP, clientPort) differs.
3. **What is the backlog in listen()?** Queue of completed-but-not-yet-accepted connections; overflow → refused/dropped SYNs (tuning matters under load spikes).
4. **Blocking vs non-blocking I/O? What is select/epoll?** → multiplexing story; why epoll beats select at scale (event-driven vs linear scan).
5. **What is the C10K problem?** Historical challenge of 10k concurrent connections — killed thread-per-connection, birthed event loops.
6. **TCP vs UDP sockets — API difference?** Stream (connect/accept, read/write on a connection) vs datagram (sendto/recvfrom, per-packet addressing).
7. **What happens if the client crashes mid-connection?** Server discovers on next write (RST/EPIPE) or via TCP keepalive/app heartbeats — silence is indistinguishable from slowness (ties to distributed-systems failure detection).
8. **Port numbers — ranges?** 0–1023 well-known (privileged), 1024–49151 registered, 49152–65535 ephemeral (client side auto-assigned).
