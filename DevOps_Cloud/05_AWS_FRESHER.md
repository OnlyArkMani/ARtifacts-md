# AWS — Fresher-Level Interview Prep

> **Tag: Theory/recall** — freshers are tested on: what each core service is (one line), the EC2/S3/IAM trio in some depth, and one architecture walkthrough.

## Cloud Basics

- **IaaS / PaaS / SaaS:** rent VMs (EC2) / rent a platform, bring code (Elastic Beanstalk, Lambda-ish) / rent finished software (Gmail).
- **Why cloud:** no upfront hardware, elastic scaling, pay-per-use, global regions, managed services (their ops team, not yours).
- **Region** (geographic area) → **Availability Zones** (isolated datacenters within it; deploy across ≥2 AZs for HA) → edge locations (CloudFront).
- **Shared responsibility model:** AWS secures the cloud (hardware, facilities); you secure what's *in* it (data, IAM, configs) — favorite question.

## The Core Services (one-liners you must have)

| Service | What it is |
|---|---|
| **EC2** | Virtual machines (instance types, AMIs, security groups, key pairs) |
| **S3** | Object storage — unlimited files in buckets, 11-nines durability, static hosting |
| **IAM** | Users, groups, roles, policies — who can do what |
| **VPC** | Your private network: subnets (public/private), route tables, internet gateway |
| **RDS** | Managed relational DBs (Postgres/MySQL) — backups, patching, Multi-AZ failover |
| **DynamoDB** | Managed NoSQL key-value at any scale (the AP store from System Design) |
| **Lambda** | Serverless functions — run code on events, no servers, pay per ms |
| **ELB/ALB** | Managed load balancers (the LB building block, productized) |
| **Auto Scaling Group** | Adds/removes EC2s by metrics — elasticity |
| **CloudFront** | CDN at AWS edge locations |
| **Route 53** | DNS (+ health-checked/geo routing) |
| **SQS / SNS** | Managed queue / pub-sub fan-out (the MQ building block) |
| **CloudWatch** | Metrics, logs, alarms |
| **EBS** | Block storage disks for EC2 (vs S3: filesystem vs objects) |

Notice: half of System Design's building blocks are AWS products — connect them explicitly in interviews.

## The Trio in Depth

**EC2:** choose AMI (image) + instance type (t3.micro…) + security group (stateful firewall: allow 22/80/443) + key pair. Pricing: on-demand / reserved-savings plans (steady loads, ~40–60% off) / **spot** (spare capacity, ~70–90% off, can be reclaimed — batch/CI).

**S3:** buckets (globally unique names) hold objects (key → bytes + metadata). Storage classes: Standard → Infrequent Access → Glacier (archive) — lifecycle rules move data down as it cools. Versioning, presigned URLs (temporary access — the upload pattern from System Design case studies), static website hosting + CloudFront.

**IAM:** **users** (humans) · **groups** (users + shared policies) · **roles** (assumed by services/temporary — *always prefer roles over embedded keys*: EC2 instance profile, Lambda execution role) · **policies** (JSON allow/deny on actions/resources). Root account: lock away, MFA, never daily-drive. Principle of least privilege.

## The Standard Architecture Walkthrough (be able to draw)

```
Route 53 (DNS) → CloudFront (CDN) → ALB (multi-AZ)
                                        │
                              Auto Scaling Group
                              [EC2] [EC2] (private subnets, AZ-a, AZ-b)
                                        │
                        RDS Multi-AZ (primary + standby)
                        + S3 (assets/uploads) + SQS (async work)
```
"Highly available web app": every layer redundant across ≥2 AZs; static content offloaded to S3+CloudFront; async work queued to SQS + worker ASG. This single diagram answers many questions.

## Most-Asked Interview Questions

1. **What is EC2 / S3 / IAM?** → one-liners + a depth detail each (pricing models, storage classes, roles-vs-users).
2. **S3 vs EBS vs EFS?** Objects via API (web-scale) vs block disk for one EC2 vs shared POSIX filesystem for many.
3. **Region vs AZ? Why deploy across AZs?** Geography vs isolated DCs; AZ failure tolerance = the basic HA answer.
4. **IAM user vs role?** Permanent identity + long-lived keys vs assumable temporary credentials — roles for services, always.
5. **Security group vs NACL?** Instance-level stateful allow-only vs subnet-level stateless allow/deny lists.
6. **What is serverless / when Lambda over EC2?** Event-driven, spiky, glue logic, zero ops vs long-running, custom runtime, steady high load (cost math flips).
7. **How do you make an app highly available on AWS?** → the walkthrough diagram; name each redundancy.
8. **RDS Multi-AZ vs read replicas?** Synchronous standby for *failover* (HA) vs async copies for *read scaling* — different problems (exactly the replication file's distinction).
9. **Public vs private subnet?** Route to internet gateway vs not (egress via NAT gateway); DBs/app servers go private.
10. **How do you avoid a surprise bill?** Budgets/alerts, right-sizing, spot for batch, lifecycle policies, kill idle resources — cost-awareness reads as seniority.
