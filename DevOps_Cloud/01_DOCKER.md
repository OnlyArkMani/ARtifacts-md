# Docker — Interview Prep

> **Tag: Theory + hands-on commands** — fresher interviews test concepts (image vs container, vs VM) plus Dockerfile literacy.

## Core Concepts

**The problem Docker solves:** "works on my machine." A container packages the app + all dependencies + OS-level libs into one runnable, portable unit that behaves identically everywhere.

**Container vs VM (the #1 question):**

| | Container | Virtual Machine |
|---|---|---|
| Virtualizes | The OS (shares host kernel) | The hardware (full guest OS) |
| Isolation via | Namespaces + cgroups (Linux kernel features) | Hypervisor |
| Size | MBs | GBs |
| Startup | Milliseconds–seconds | Minutes |
| Density | 100s per host | ~10s per host |
| Isolation strength | Process-level (weaker) | Full (stronger) |

Underneath: **namespaces** isolate what a process *sees* (PIDs, network, filesystem, users); **cgroups** limit what it *uses* (CPU, memory, I/O). A container is just a well-isolated Linux process — saying this shows real understanding.

**Image vs container:** image = read-only template (class); container = running instance with a thin writable layer (object). Images are built from **layers** — each Dockerfile instruction creates one; layers are cached and shared between images (why builds are fast and storage efficient).

## Dockerfile Literacy

```dockerfile
FROM node:20-alpine            # base image (small!)
WORKDIR /app
COPY package*.json ./          # copy deps manifest FIRST →
RUN npm ci                     # this layer caches until deps change
COPY . .                       # app code changes often → later layer
EXPOSE 3000
USER node                      # don't run as root
CMD ["node", "server.js"]      # default command (exec form)
```

Key points interviewers probe:
- **Layer-cache ordering:** least-changing instructions first; copying code before `npm ci` busts the dependency cache every build.
- **CMD vs ENTRYPOINT:** CMD = default args (overridable at `docker run`); ENTRYPOINT = fixed executable; combined: ENTRYPOINT + CMD-as-default-flags.
- **Multi-stage builds:** build in a fat image, copy artifacts into a slim runtime image → small, tool-free production images:

```dockerfile
FROM golang:1.22 AS build
WORKDIR /src
COPY . . 
RUN go build -o /bin/app
FROM alpine
COPY --from=build /bin/app /app
CMD ["/app"]
```

## Essential Commands (be fluent)

`docker build -t app:v1 .` · `docker run -d -p 8080:3000 --name web app:v1` · `docker ps` / `logs -f web` / `exec -it web sh` · `docker images` / `pull` / `push` · `docker stop && docker rm` · `docker system prune`.

**Networking:** default bridge; user-defined bridge gives container-name DNS (`db:5432`); `-p host:container` publishes ports. **Volumes:** containers are ephemeral — data outlives them via named volumes (`-v pgdata:/var/lib/postgresql/data`) or bind mounts (dev). **docker-compose:** declarative multi-container dev environments (`services: web, db, cache` + one `docker compose up`).

## Most-Asked Interview Questions

1. **Container vs VM?** → table + kernel-sharing point.
2. **Image vs container?** Template vs running instance (+ writable layer).
3. **What are layers / why is ordering important in a Dockerfile?** Caching: unchanged prefix of instructions is reused; order least→most volatile.
4. **CMD vs ENTRYPOINT?** Default-overridable vs fixed executable.
5. **How do containers persist data?** Volumes/bind mounts; container FS is throwaway.
6. **How do two containers talk?** Same user-defined network → DNS by service/container name; compose does this automatically.
7. **What is a registry?** Image store (Docker Hub, ECR, GCR): build → push → pull → run pipeline backbone.
8. **Multi-stage builds — why?** Small final images (no compilers/deps), fewer CVEs, faster pulls.
9. **Why not run as root in a container?** Kernel is shared — container escape = host root; `USER`, read-only FS, minimal base images (distroless/alpine).
10. **`docker exec` vs `docker run`?** Command in an *existing* container vs a *new* container.
