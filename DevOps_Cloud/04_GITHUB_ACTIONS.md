# GitHub Actions — Interview Prep

> **Tag: Hands-on YAML literacy** — the modern default CI/CD for anything on GitHub; expect to read/write workflow files and compare with Jenkins.

## Core Model (five words each)

- **Workflow:** a YAML file in `.github/workflows/` — one automation.
- **Event/trigger:** what starts it (`push`, `pull_request`, `schedule`, `workflow_dispatch` = manual).
- **Job:** group of steps on one **runner** (fresh VM: ubuntu/windows/macos, or self-hosted). Jobs run in **parallel** unless `needs:` chains them.
- **Step:** a shell command (`run:`) or a reusable **action** (`uses:`).
- **Action:** packaged step from the marketplace (`actions/checkout@v4`, `actions/setup-node@v4`).

## A Real Workflow (read/write level)

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix: { node: [18, 20] }        # run job per version, in parallel
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: ${{ matrix.node }}, cache: npm }
      - run: npm ci
      - run: npm test

  deploy:
    needs: test                          # only after test passes
    if: github.ref == 'refs/heads/main'  # only on main
    runs-on: ubuntu-latest
    environment: production              # protection rules/approvals
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t app:${{ github.sha }} .
      - run: echo "${{ secrets.REGISTRY_TOKEN }}" | docker login -u me --password-stdin
      - run: docker push app:${{ github.sha }}
```

Talking points: `on:` filters (branches, paths), matrix builds, `needs` for ordering, `if:` conditions, **`secrets.*`** (encrypted, masked in logs), `environment` protection (required reviewers before prod deploys), caching (`actions/cache` / setup-* built-ins) for speed, `${{ github.sha }}` context variables.

## GitHub Actions vs Jenkins (the guaranteed comparison)

| | GitHub Actions | Jenkins |
|---|---|---|
| Hosting | SaaS (GitHub-hosted runners) or self-hosted runners | Self-hosted server (you operate it) |
| Config | YAML in repo (only) | Jenkinsfile (or legacy UI jobs) |
| Setup | Zero — enabled per repo | Install, plugins, agents, upkeep |
| Ecosystem | Marketplace actions | Plugins (older, vaster) |
| Pricing | Free tier; pay per minute beyond | Free software; you pay infra/ops |
| Fit | GitHub-based teams, standard pipelines | Complex/legacy/on-prem, deep customization |

One-liner: *Actions is CI/CD as a product; Jenkins is CI/CD as a platform you run.*

## Beyond CI (worth name-dropping)

Scheduled jobs (`cron`), issue/PR automation (labelers, stale bots), release publishing, **reusable workflows** (`workflow_call` — org-wide DRY), **OIDC to cloud** (deploy to AWS without long-lived keys — modern best practice), concurrency groups (cancel superseded runs on the same PR).

## Most-Asked Interview Questions

1. **Workflow vs job vs step vs action?** → the four-liner model; jobs parallel by default, steps sequential.
2. **How do you run jobs in order?** `needs:`; matrix for fan-out; `if:` for conditions.
3. **Where do secrets live?** Repo/org/environment secrets → `${{ secrets.X }}`, masked; environment-level adds approval gates.
4. **Actions vs Jenkins — which and why?** → table; answer with context (team size, GitHub usage, ops capacity).
5. **What triggers exist besides push?** PR, schedule/cron, manual dispatch (with inputs), release, workflow_call from other workflows.
6. **How do you speed up a slow pipeline?** Dependency caching, matrix/job parallelism, path filters (skip unaffected), concurrency-cancel stale runs, smaller Docker layers.
7. **GitHub-hosted vs self-hosted runners?** Zero-ops ephemeral VMs vs your hardware (GPU, network access to private infra, cost control) — security caution: self-hosted + public repo PRs = code-execution risk (classic gotcha).
8. **How would you deploy to AWS from Actions?** OIDC federation → temporary credentials → deploy step; no stored keys (cross-ref `05_AWS_FRESHER.md`).
