# Jenkins — Interview Prep

> **Tag: Theory + pipeline-code literacy** — know CI/CD concepts first, Jenkins mechanics second, and be able to read/write a basic Jenkinsfile.

## CI/CD First (the context Jenkins lives in)

- **Continuous Integration:** every push → automatic build + tests → broken code caught in minutes, integration pain amortized.
- **Continuous Delivery:** every green build → automatically packaged and *deployable* (human clicks deploy).
- **Continuous Deployment:** green build → *automatically deployed* to production, no human.

A typical pipeline: `checkout → build → unit tests → static analysis → package (Docker image) → push registry → deploy staging → integration tests → deploy prod (gate/auto)`.

## Jenkins Core Concepts

- **What it is:** self-hosted, open-source automation server; the long-time default CI/CD engine; plugin ecosystem (~everything via plugins).
- **Architecture:** **controller** (schedules, UI, config) + **agents/nodes** (execute jobs — scale out, isolate environments, run different OSes). Distributed builds = the controller/agent question.
- **Job types:** Freestyle (UI-configured, legacy) vs **Pipeline** (code-defined — the modern answer).
- **Jenkinsfile:** pipeline-as-code, versioned with the repo — reviewable, reproducible, survives Jenkins box loss. Declarative (structured, preferred) vs scripted (Groovy, flexible) syntax.
- **Triggers:** webhook on push (preferred), poll SCM, cron, upstream job.
- **Shared libraries:** reusable pipeline functions across repos (DRY for pipelines).

## A Declarative Jenkinsfile (read/write level)

```groovy
pipeline {
    agent any                          // or: agent { docker { image 'node:20' } }
    environment { REGISTRY = 'myreg.io' }
    stages {
        stage('Build')  { steps { sh 'npm ci && npm run build' } }
        stage('Test')   { steps { sh 'npm test' }
                          post { always { junit 'reports/**/*.xml' } } }
        stage('Docker') { steps { sh 'docker build -t $REGISTRY/app:$BUILD_NUMBER .'
                                  sh 'docker push $REGISTRY/app:$BUILD_NUMBER' } }
        stage('Deploy') {
            when { branch 'main' }                 // conditional stage
            steps { sh 'kubectl set image deploy/web web=$REGISTRY/app:$BUILD_NUMBER' }
        }
    }
    post {
        failure { mail to: 'team@x.com', subject: "Build ${BUILD_NUMBER} failed" }
    }
}
```

Talking points in this snippet: stages/steps structure, `agent` (incl. Docker agents for clean environments), `environment`, `when` conditions, `post` blocks (always/success/failure), `$BUILD_NUMBER` for image tagging, credentials via `credentials()` helper (never hardcode secrets).

## Most-Asked Interview Questions

1. **What is CI/CD? Delivery vs deployment?** → definitions above; the human-gate distinction.
2. **What is Jenkins and why use it?** Automation server executing pipelines on triggers; free, self-hosted, plugin-rich; contrast: GitHub Actions/GitLab CI are hosted+YAML (see `04_GITHUB_ACTIONS.md`).
3. **Explain controller vs agent.** Scheduling brain vs execution workers; scale + environment isolation; don't run builds on the controller (security/performance).
4. **Freestyle vs Pipeline job?** UI-clicked config vs code in repo — versioning, review, reuse win.
5. **Declarative vs scripted pipeline?** Structured DSL with validation vs raw Groovy power; declarative default.
6. **How does Jenkins know code was pushed?** Webhook from GitHub/GitLab (push-based, instant) vs SCM polling (pull-based, wasteful).
7. **How do you handle secrets in Jenkins?** Credentials store + `credentials()` binding; masked in logs; never in the Jenkinsfile.
8. **What are build artifacts?** Outputs (jars, images, reports) archived per build / pushed to registries (Nexus/Artifactory/ECR).
9. **How would you set up CI for a new repo?** Jenkinsfile in repo → multibranch pipeline job → webhook → PR builds + main deploys — concise end-to-end answer.
10. **Blue-green vs rolling vs canary deployment?** Swap-entire-environment / gradual replace / percentage-based trial — pipeline stages implement whichever; ties to K8s rollout strategies.
