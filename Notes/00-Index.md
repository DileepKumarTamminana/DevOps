# DevOps Skills — Topic-wise Notes (From Basics)

A structured learning path covering the core DevOps skills. Start at the top and work down — each file is self-contained and builds from fundamentals.

## Reading order

1. [01 — CI/CD & Automation](01-CICD-and-Automation.md)
   - What CI/CD is, GitHub Actions, YAML pipelines, GitHub Copilot
2. [02 — Cloud Platforms](02-Cloud-Platforms.md)
   - Cloud basics, Microsoft Azure (AZ-900 / DP-900), AWS, Azure DevOps
3. [03 — Infrastructure & Configuration](03-Infrastructure-and-Config.md)
   - Docker, Linux administration, environment configuration, release management
4. [04 — Monitoring & ITSM](04-Monitoring-and-ITSM.md)
   - ServiceNow, incident management, change management, SLA tracking
5. [05 — Scripting & Query](05-Scripting-and-Query.md)
   - Bash, PowerShell, SQL
6. [06 — Practices](06-Practices.md)
   - Production support (L2/L3), root cause analysis, post-release validation

### Apply & test your knowledge
7. [07 — Hands-On Projects](07-HandsOn-Projects.md)
   - 8 simple, runnable projects from first script to full CI/CD + cloud deploy
8. [09 — Projects: Step-by-Step](09-Projects-StepByStep.md)
   - Click-by-click execution guide for every project (Windows/PowerShell) with troubleshooting
9. [08 — Interview Practice](08-Interview-Practice.md)
   - Topic Q&A, scenarios, resume-defence questions, rapid-fire, 1-week plan

## How to use these notes

- Each topic starts with **"What it is"** and **"Why it matters"** before going into mechanics.
- Code/command blocks are runnable examples — try them in a safe sandbox.
- "Interview-ready" callouts summarise points you should be able to explain out loud.
- Cross-references use `[[links]]` between related topics.

## The big picture: what is DevOps?

**DevOps** = a culture + set of practices that shortens the gap between writing code
(**Dev**) and running it reliably in production (**Ops**).

The goal: ship software **faster**, **more frequently**, and **more safely**.

Core pillars:

| Pillar | Meaning | Tools you use |
|---|---|---|
| **Automation** | Remove manual, error-prone steps | GitHub Actions, scripts |
| **CI (Continuous Integration)** | Merge & test code often | GitHub Actions, Azure DevOps |
| **CD (Continuous Delivery/Deployment)** | Automate releases | Pipelines, approvals |
| **Infrastructure as Code (IaC)** | Manage infra via files, not clicks | Terraform, ARM/Bicep |
| **Monitoring & Feedback** | Know when things break | ServiceNow, dashboards |
| **Collaboration** | Dev + QA + Ops work as one | Branching, reviews, ITSM |

The classic loop: **Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (back to Plan)**.
