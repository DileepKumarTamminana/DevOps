# 02 — Cloud Platforms

Covers: Cloud basics · Microsoft Azure (AZ-900, DP-900) · AWS · Azure DevOps

---

## 1. Cloud computing fundamentals

**Cloud computing** = renting computing resources (servers, storage, databases,
networking) over the internet, pay-as-you-go, instead of buying and running your own
hardware.

### The three service models
| Model | You manage | Provider manages | Example |
|---|---|---|---|
| **IaaS** (Infrastructure) | OS, apps, data | Hardware, virtualization | Azure VM, AWS EC2 |
| **PaaS** (Platform) | Apps, data | OS, runtime, hardware | Azure App Service, AWS Elastic Beanstalk |
| **SaaS** (Software) | Just use it | Everything | Office 365, ServiceNow |

Mnemonic: *IaaS = you cook with raw ingredients; PaaS = meal kit; SaaS = restaurant.*

### Deployment models
- **Public cloud** — shared infrastructure (Azure, AWS).
- **Private cloud** — dedicated to one org.
- **Hybrid** — mix of on-prem and public cloud (common in life sciences/regulated industries).

### Core cloud benefits (the "why")
- **Elasticity / scalability** — scale up/down on demand.
- **Pay-as-you-go** — turn capital expense (CapEx) into operating expense (OpEx).
- **High availability & global reach** — regions and availability zones.
- **Reduced maintenance** — provider handles the physical layer.

> **Interview-ready:** "IaaS gives me VMs to manage; PaaS lets me deploy apps without
> managing the OS; SaaS is ready-to-use software. Cloud's main wins are elasticity and
> the CapEx-to-OpEx shift."

---

## 2. Microsoft Azure

### AZ-900 (Azure Fundamentals) — the essentials

**Global infrastructure:**
- **Region** — a geographic area with datacenters (e.g. East US).
- **Availability Zone** — physically separate datacenters within a region (for fault tolerance).
- **Resource Group** — a logical container for related resources; you manage them together.
- **Subscription** — billing + access boundary; sits under a Management Group.

**Core services:**
| Category | Service | Purpose |
|---|---|---|
| Compute | **Virtual Machines** | IaaS servers |
| Compute | **App Service** | PaaS web app hosting |
| Compute | **Azure Functions** | Serverless event-driven code |
| Compute | **AKS** | Managed Kubernetes |
| Storage | **Blob Storage** | Object storage (files, backups) |
| Storage | **Azure Files** | Managed file shares |
| Network | **Virtual Network (VNet)** | Private network in Azure |
| Network | **Load Balancer / App Gateway** | Distribute traffic |

**Identity, governance & cost:**
- **Microsoft Entra ID** (formerly Azure AD) — identity & access management.
- **RBAC (Role-Based Access Control)** — assign roles (Owner, Contributor, Reader)
  at a scope (subscription/RG/resource). Principle of **least privilege**.
- **Azure Policy** — enforce rules (e.g. "only allowed regions").
- **Cost Management** — budgets, alerts, cost analysis. ("Cost monitoring" on your resume.)
- **Pricing Calculator / TCO Calculator** — estimate costs.

### DP-900 (Azure Data Fundamentals) — the essentials
Focuses on data concepts:
- **Relational data** — structured, tables, SQL. Azure SQL Database, Azure SQL Managed Instance.
- **Non-relational (NoSQL)** — Azure Cosmos DB (key-value, document, graph).
- **Analytics** — Azure Synapse Analytics, data warehouses, ETL/ELT pipelines.
- **OLTP vs OLAP** — transactional (day-to-day writes) vs analytical (reporting/queries).
- **Data roles** — Database Administrator, Data Engineer, Data Analyst.

> **Interview-ready:** "A resource group is a logical container; RBAC controls who can do
> what at a given scope. For data, I know the relational vs NoSQL distinction and OLTP vs OLAP."

---

## 3. AWS (Amazon Web Services)

AWS is the largest cloud provider. The naming differs but concepts map closely to Azure.

### Core services & their Azure equivalents
| Category | AWS | Azure equivalent |
|---|---|---|
| Compute (VM) | **EC2** | Virtual Machines |
| Serverless | **Lambda** | Functions |
| Object storage | **S3** | Blob Storage |
| Networking | **VPC** | Virtual Network |
| Identity | **IAM** | Entra ID + RBAC |
| Managed DB | **RDS** | Azure SQL |
| Containers (k8s) | **EKS** | AKS |
| DNS | **Route 53** | Azure DNS |
| IaC | **CloudFormation** | ARM / Bicep |

### AWS DevOps-relevant services
- **IAM** — users, groups, roles, policies (JSON). Least-privilege is critical.
- **S3** — durable object storage; used for artifacts, logs, static sites, backups.
- **CodePipeline / CodeBuild / CodeDeploy** — AWS's native CI/CD suite.
- **CloudWatch** — monitoring, logs, metrics, alarms.
- **CloudFormation** — Infrastructure as Code (templates in YAML/JSON).

> **Interview-ready:** "EC2 ≈ Azure VM, S3 ≈ Blob, IAM ≈ Entra+RBAC. AWS has its own CI/CD
> tools (CodePipeline/Build/Deploy) but I can also drive AWS deployments from GitHub Actions."

---

## 4. Azure DevOps (the platform)

Not to be confused with "DevOps" the practice. **Azure DevOps** is Microsoft's
all-in-one suite (alternative/complement to GitHub):

| Service | Purpose |
|---|---|
| **Azure Boards** | Work items, backlogs, sprints (Agile/Scrum/Kanban) |
| **Azure Repos** | Git repositories |
| **Azure Pipelines** | CI/CD (YAML or classic editor) |
| **Azure Artifacts** | Package feeds (NuGet, npm, Maven) |
| **Azure Test Plans** | Manual & exploratory testing |

### Azure Pipelines YAML (compare to GitHub Actions)
```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - script: npm ci && npm run build
            displayName: 'Build app'

  - stage: Deploy
    dependsOn: Build
    jobs:
      - deployment: DeployProd
        environment: 'Production'   # supports approvals & checks
        strategy:
          runOnce:
            deploy:
              steps:
                - script: ./deploy.sh
```
Key terms: **trigger** (event), **pool** (runner/agent), **stages → jobs → steps**,
**environments** (with approval checks), **variable groups** (shared/secret variables),
**service connections** (auth to Azure/AWS).

> **Interview-ready:** "Azure DevOps Pipelines uses stages → jobs → steps with agent pools.
> The concepts mirror GitHub Actions; environments provide deployment approval gates in both."

---

## Practice checklist
- [ ] Create a resource group and deploy a VM or App Service in Azure (free tier).
- [ ] Assign a Reader role to a user via RBAC.
- [ ] Set a cost budget + alert.
- [ ] Map 5 AWS services to their Azure equivalents from memory.
- [ ] Write a minimal Azure Pipelines YAML with a Build and Deploy stage.
