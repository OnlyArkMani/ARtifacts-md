# DevOps & Cloud — Index

## Study Order & Priorities

1. `01_DOCKER.md` — **Tier 1.** Container vs VM + image/layer model; prerequisite for K8s.
2. `05_AWS_FRESHER.md` — **Tier 1** for any cloud-touching role. EC2/S3/IAM trio + the HA architecture diagram.
3. `04_GITHUB_ACTIONS.md` — **Tier 2.** The modern CI/CD default; YAML literacy.
4. `02_KUBERNETES.md` — **Tier 2** (Tier 1 for DevOps-titled roles). Object-model ladder: Pod→Deployment→Service.
5. `03_JENKINS.md` — **Tier 2/3.** CI/CD concepts transfer from Actions; Jenkins specifics for enterprises that use it.

## Cross-Cutting Comparisons (fast scan)

**Container vs VM:** shared kernel + namespaces/cgroups, MBs, ms-startup vs hypervisor + guest OS, GBs, minutes.

**Image vs container:** template vs running instance. **CMD vs ENTRYPOINT:** overridable default vs fixed executable.

**Docker vs Kubernetes:** runs containers on one host vs orchestrates them across a cluster (scheduling, healing, scaling, discovery).

**Pod vs Deployment vs Service:** instance group / replica+rollout manager / stable network front. **Liveness vs readiness:** restart vs traffic-gate.

**Jenkins vs GitHub Actions:** self-hosted platform (plugins, Groovy, you operate) vs hosted product (YAML, marketplace, zero setup).

**CI vs CD vs CD:** merge-and-test always → deployable always → deployed always.

**S3 vs EBS vs EFS:** objects / block / shared filesystem. **SG vs NACL:** instance-stateful vs subnet-stateless. **User vs role:** permanent vs assumed-temporary. **Multi-AZ vs read replica:** failover vs read scaling.

## Ties to Your Other Artifacts
K8s Services/Ingress = the Load Balancer & API Gateway building blocks · SQS/SNS = Message Queues file · DynamoDB = the Key-Value Store case study, productized · CloudFront = CDN file · etcd = CP consensus from CAP file · deployment strategies (rolling/canary/blue-green) pair with System Design's zero-downtime discussions.
