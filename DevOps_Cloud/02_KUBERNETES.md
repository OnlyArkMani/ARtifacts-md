# Kubernetes — Interview Prep

> **Tag: Theory-heavy** — freshers are tested on the object model (Pod→Deployment→Service), architecture, and the "desired state" philosophy, not cluster ops.

## The Core Idea

Docker runs containers on ONE machine. Kubernetes (K8s) runs containers across a **cluster**: scheduling onto nodes, restarting failures, scaling replicas, load-balancing traffic, rolling out updates. You declare **desired state** in YAML; controllers reconcile actual → desired continuously (**the reconciliation loop** — K8s's single central concept: kill a pod and it resurrects, because 3 replicas is the declared truth).

## Architecture

```
CONTROL PLANE                        WORKER NODES
┌─────────────────────────┐          ┌──────────────────────┐
│ API server  (front door)│◀────────▶│ kubelet (node agent) │
│ etcd        (state DB)  │          │ kube-proxy (routing) │
│ scheduler   (pod→node)  │          │ container runtime    │
│ controller-manager      │          │   [Pod] [Pod] [Pod]  │
│  (reconciliation loops) │          └──────────────────────┘
└─────────────────────────┘          (× N nodes)
```

Everything talks through the **API server**; **etcd** stores all cluster state (a CP consensus store — Raft; ties to System Design); **scheduler** places pods by resources/affinity; **kubelet** makes containers on its node match assignments.

## The Object Model (the testable ladder)

- **Pod:** smallest unit — one or more tightly-coupled containers sharing network (localhost) and volumes. Usually 1 app container (+ sidecars). Pods are **mortal** — never create bare pods in production.
- **ReplicaSet:** keeps N pod copies alive.
- **Deployment:** manages ReplicaSets + **rolling updates/rollbacks** — your default workload object.
- **Service:** stable virtual IP + DNS name in front of ephemeral pods (selected by **labels**). Types: `ClusterIP` (internal), `NodePort` (dev/external via node ports), `LoadBalancer` (cloud LB). Services = K8s's built-in service discovery + L4 load balancing.
- **Ingress:** L7 HTTP routing (host/path → service) + TLS — one entry point for many services (the API-gateway shape).
- **ConfigMap / Secret:** config and credentials injected as env vars/files — decoupled from images.
- **Namespace:** virtual sub-clusters (envs, teams, quotas).
- Others to name-drop: StatefulSet (stable identity/storage — DBs), DaemonSet (one per node — log agents), Job/CronJob (batch/scheduled), HPA (autoscaling on metrics), PV/PVC (storage abstraction).

## Minimal YAML (recognize/write)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: web }
spec:
  replicas: 3
  selector: { matchLabels: { app: web } }
  template:
    metadata: { labels: { app: web } }
    spec:
      containers:
      - name: web
        image: myapp:v2
        ports: [{ containerPort: 8080 }]
        resources:
          requests: { cpu: 100m, memory: 128Mi }   # scheduling basis
          limits:   { cpu: 500m, memory: 256Mi }   # hard cap
        readinessProbe: { httpGet: { path: /health, port: 8080 } }
---
apiVersion: v1
kind: Service
metadata: { name: web-svc }
spec:
  selector: { app: web }        # ← label matching wires it up
  ports: [{ port: 80, targetPort: 8080 }]
```

## Key Behaviors to Explain

- **Rolling update:** new ReplicaSet scales up as old scales down (maxSurge/maxUnavailable); `kubectl rollout undo` reverts. Zero-downtime deploys via readiness probes gating traffic.
- **Liveness vs readiness probes:** liveness fails → restart container; readiness fails → remove from Service endpoints (no restart). Mixing them up causes restart storms — classic question.
- **Requests vs limits:** requests = scheduler's math; limits = runtime cap (CPU throttled, memory **OOMKilled**).
- **Self-healing walk-through:** node dies → controller sees missing replicas → scheduler places new pods on healthy nodes → service endpoints update — narrate this chain; it demonstrates the whole system.

## kubectl fluency

`kubectl get pods/deploy/svc` · `describe pod X` (events = debugging goldmine) · `logs -f X` · `exec -it X -- sh` · `apply -f app.yaml` · `rollout status/undo deploy/web` · `scale deploy/web --replicas=5`.

## Most-Asked Interview Questions

1. **Why Kubernetes over plain Docker?** Multi-node scheduling, self-healing, scaling, rolling deploys, service discovery — Docker runs containers, K8s *operates* them.
2. **What is a Pod? Why not just containers?** Co-scheduled container group sharing net/volumes; the atomic scheduling unit; sidecar pattern.
3. **Pod vs Deployment vs Service?** Instance / manager of replicas+updates / stable network front. The ladder answer.
4. **Deployment vs StatefulSet?** Interchangeable replicas vs stable identities + ordered startup + per-pod storage (databases).
5. **Liveness vs readiness?** Restart vs traffic-gate — and the failure mode of confusing them.
6. **How does a rolling update work? Rollback?** Two ReplicaSets crossfade; history retained → undo.
7. **What happens when a node fails?** → the self-healing chain above.
8. **ClusterIP vs NodePort vs LoadBalancer vs Ingress?** Internal VIP / node ports / cloud L4 / shared L7 router.
9. **ConfigMap vs Secret?** Plain config vs base64-obscured sensitive data (+ RBAC/encryption-at-rest for real security).
10. **What is etcd and why does it matter?** Consensus-backed source of truth; lose quorum → control plane read-only (ties to CAP/CP).
