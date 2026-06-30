# 13 — Glossary & Acronyms (DevOps A–Z)

A fast lookup for the terms used across these notes. DevOps is acronym-heavy — being able to
expand and define these on the spot is half of interview confidence.

> Tip: cover the right column and try to define each from the term alone.

---

## Acronyms (expand them instantly)

| Acronym | Expansion | One-line meaning |
|---|---|---|
| **CI** | Continuous Integration | Merge & auto-test code frequently |
| **CD** | Continuous Delivery / Deployment | Automate release (gated / fully automatic) |
| **IaC** | Infrastructure as Code | Manage infra via versioned files (Terraform, ARM) |
| **VCS** | Version Control System | Tracks code history (Git) |
| **PR** | Pull Request | Propose + review a branch merge |
| **SCM** | Source Code Management | Same space as VCS/Git hosting |
| **YAML** | YAML Ain't Markup Language | Indentation-based config format |
| **JSON** | JavaScript Object Notation | Data format (configs, IAM policies) |
| **API** | Application Programming Interface | Contract for talking to a service |
| **REST** | Representational State Transfer | HTTP-based API style |
| **CLI** | Command-Line Interface | Text command tool (az, aws, kubectl) |
| **VM** | Virtual Machine | Virtualised server (IaaS) |
| **OS** | Operating System | Linux/Windows |
| **IaaS / PaaS / SaaS** | Infrastructure / Platform / Software as a Service | Cloud service models |
| **CapEx / OpEx** | Capital / Operating Expenditure | Upfront vs pay-as-you-go cost |
| **RBAC** | Role-Based Access Control | Permissions via roles at a scope |
| **IAM** | Identity & Access Management | Who can do what (AWS/Azure) |
| **VNet / VPC** | Virtual Network / Virtual Private Cloud | Private cloud network |
| **AZ** | Availability Zone | Isolated datacenter within a region |
| **ECR / ACR** | Elastic / Azure Container Registry | Private image registries |
| **AKS / EKS / GKE** | Azure / Elastic / Google Kubernetes Service | Managed Kubernetes |
| **K8s** | Kubernetes | Container orchestrator (8 letters between K and s) |
| **HPA** | Horizontal Pod Autoscaler | Auto-scales Pods in K8s |
| **PVC** | Persistent Volume Claim | Storage request in K8s |
| **DNS** | Domain Name System | Maps names → IP addresses |
| **TLS / SSL** | Transport Layer Security | Encrypts traffic (HTTPS) |
| **ITSM** | IT Service Management | Practices for running IT services |
| **ITIL** | IT Infrastructure Library | The main ITSM framework |
| **CMDB** | Configuration Management Database | Inventory of CIs & relationships |
| **CI (ITIL)** | Configuration Item | An asset (server/app/DB) — *not* Continuous Integration! |
| **INC / CHG / PRB** | Incident / Change / Problem | ServiceNow record types |
| **RFC** | Request for Change | A proposed change |
| **CAB / ECAB** | (Emergency) Change Advisory Board | Approves changes |
| **SLA / OLA** | Service Level / Operational Level Agreement | External / internal commitments |
| **KPI** | Key Performance Indicator | A measured metric |
| **MTTR** | Mean Time To Resolve / Recover | Avg time to fix an incident |
| **MTBF** | Mean Time Between Failures | Avg uptime between failures |
| **RTO / RPO** | Recovery Time / Point Objective | Max downtime / max data loss tolerated |
| **RCA** | Root Cause Analysis | Find the underlying cause |
| **UAT** | User Acceptance Testing | Business/QA validation env |
| **L1 / L2 / L3** | Support tiers | First-line → expert/engineering |
| **DR** | Disaster Recovery | Restoring service after a major failure |
| **SemVer** | Semantic Versioning | `MAJOR.MINOR.PATCH` |
| **DAG** | Directed Acyclic Graph | Job dependency graph (pipelines) |
| **SRE** | Site Reliability Engineering | Ops via software-engineering practices |
| **SLO / SLI** | Service Level Objective / Indicator | Target / measured signal behind an SLA |

---

## Glossary (concepts)

**Artifact** — a built, deployable output (image, zip, package); built once, promoted across
environments.

**Blast radius** — how much is affected if something fails; canary/blue-green limit it.

**Blue-Green deployment** — two identical environments; switch traffic to the new one for
instant rollback. See [[03-Infrastructure-and-Config]].

**Canary deployment** — release to a small % of users first, then ramp up.

**Container** — app + dependencies packaged to run identically anywhere; shares the host
kernel. See [[03-Infrastructure-and-Config]].

**Declarative vs Imperative** — declarative = state *what* you want (K8s manifest, Terraform);
imperative = list *how* (shell commands). DevOps favours declarative.

**Drift** — when real infrastructure diverges from what IaC says it should be.

**Idempotent** — running an operation multiple times gives the same result as once (key for
safe automation/IaC).

**Immutable infrastructure** — don't patch servers in place; replace them with new
versions (rebuild the image, redeploy).

**Least privilege** — grant the minimum access needed; core security principle (RBAC/IAM).

**Observability** — ability to understand system state from its outputs: **metrics, logs,
traces** (the "three pillars"). Broader than monitoring.

**Orchestration** — automated management of many containers (scheduling, scaling, healing) —
Kubernetes. See [[12-Kubernetes-Basics]].

**Pipeline-as-code** — the build/deploy process defined in versioned files (e.g.
`.github/workflows`). See [[01-CICD-and-Automation]].

**Reconciliation loop** — controller continuously drives actual state toward desired state
(Kubernetes core idea).

**Rollback** — revert to a previously known-good version when a release fails.

**Rolling update** — replace instances in batches so there's no full downtime.

**Runbook** — step-by-step guide to handle a known operational task/incident; reduces MTTR.

**Self-healing** — system automatically replaces failed components.

**Shift-left** — move testing/security earlier in the pipeline to catch issues sooner.

**Smoke test** — quick check that critical paths work after a deploy. See [[06-Practices]].

**Stateless vs Stateful** — stateless keeps no client data between requests (easy to scale);
stateful persists data (needs volumes/DBs).

**Toil** — repetitive manual operational work; the thing automation eliminates.

**Webhook** — an HTTP callback fired on an event (e.g. a `git push` triggers a pipeline).

**Zero-downtime deployment** — releasing without interrupting users (blue-green / rolling /
canary + health checks).

---

## The three "CI"s that get confused
- **CI** = Continuous **Integration** (CI/CD context).
- **CI** = Configuration **Item** (ITIL/ServiceNow context — a tracked asset).
- **CI** = the abbreviation people also use loosely for the whole "pipeline."
> Always read which world you're in. In a ServiceNow sentence, CI = Configuration Item.

---

## Quick "expand this acronym" drill
Cover the table and rattle these off: CI/CD, IaC, RBAC, MTTR, RCA, SLA vs OLA, RTO vs RPO,
AKS/EKS, CMDB, CAB, UAT, SemVer. If any made you pause, that's your next revision target.
