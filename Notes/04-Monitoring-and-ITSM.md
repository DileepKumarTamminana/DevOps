# 04 — Monitoring & ITSM

Covers: ServiceNow · Incident management · Change management · SLA tracking

ITSM = **IT Service Management**: the practices for delivering and supporting IT services.
The most common framework is **ITIL** (IT Infrastructure Library). ServiceNow is the most
common ITSM *tool*.

---

## 1. ITIL basics (the framework behind the terms)

ITIL defines standard processes so IT teams handle work consistently. The processes you
use most as a DevOps/support engineer:
- **Incident Management** — restore service fast when something breaks.
- **Problem Management** — find and fix the *root cause* of recurring incidents.
- **Change Management** — control changes to reduce risk.
- **Request Fulfilment** — handle standard service requests (access, new VM, etc.).

> Key distinction: an **Incident** is an *unplanned interruption* ("the site is down").
> A **Problem** is the *underlying cause* of one or more incidents. A **Change** is a
> *planned modification* to the environment.

---

## 2. ServiceNow

**What it is:** a cloud (SaaS) platform that runs ITSM workflows. It's where tickets
(incidents, changes, requests) live and move through their lifecycle.

### Core record types ("tables")
| Record | Prefix | Purpose |
|---|---|---|
| **Incident** | INC | Something is broken |
| **Change Request** | CHG | A planned change |
| **Problem** | PRB | Root-cause investigation |
| **Service Request** | REQ / RITM | A standard request from a catalog |
| **Configuration Item (CI)** | — | An asset (server, app, DB) in the **CMDB** |

### Key concepts
- **CMDB (Configuration Management Database)** — inventory of all CIs and their
  relationships; used for impact analysis.
- **Assignment groups** — teams that tickets get routed to (e.g. "L2 App Support").
- **Workflow / state model** — tickets move through states: New → In Progress →
  Resolved → Closed.
- **SLAs** — ServiceNow tracks time targets on tickets and flags breaches.
- **Knowledge Base (KB)** — documented solutions/runbooks.

> **Interview-ready:** "In ServiceNow I work incidents (INC) and changes (CHG), tickets are
> routed via assignment groups, CIs live in the CMDB, and SLA timers track response and
> resolution targets against each ticket."

---

## 3. Incident Management

**Goal:** restore normal service **as quickly as possible** and minimise business impact —
*not* necessarily to find the root cause (that's Problem Management).

### The lifecycle
```
Detect → Log → Categorize → Prioritize → Assign → Investigate/Diagnose
      → Resolve → Recover → Close → (Post-incident review)
```

### Priority = Impact × Urgency
| | High Urgency | Low Urgency |
|---|---|---|
| **High Impact** | P1 (Critical) | P2 (High) |
| **Low Impact** | P3 (Medium) | P4 (Low) |
- **Impact** = how many users/how much business affected.
- **Urgency** = how quickly it needs fixing.
- **Priority** (P1–P4) drives the SLA timer and escalation.

### Support tiers (L1/L2/L3)
| Tier | Role |
|---|---|
| **L1** | First response; triage, known fixes, log tickets |
| **L2** | Deeper technical troubleshooting (your level) |
| **L3** | Specialists/engineering; code-level or infra-deep issues |

- **Escalation** — moving a ticket up (L2→L3) or to management when it can't be resolved
  in time or scope.
- **MTTR (Mean Time To Resolve/Recover)** — average time to fix; a key support KPI.
  Lowering MTTR (e.g. via runbooks/automation) is a strong achievement to cite.

> **Interview-ready:** "Incident management is about fast service restoration. Priority is
> Impact × Urgency, which sets the SLA. I work L2/L3 incidents, escalate to L3 when needed,
> and we track MTTR as the headline metric."

---

## 4. Change Management

**Goal:** make changes to production in a **controlled** way to minimise risk of outages.

### Types of change
| Type | Description | Approval |
|---|---|---|
| **Standard** | Low-risk, pre-approved, repeatable (e.g. routine patch) | Pre-authorised |
| **Normal** | Needs assessment + approval (most changes) | CAB / approvers |
| **Emergency** | Urgent fix for a major incident | Expedited (ECAB) |

### Key concepts
- **CAB (Change Advisory Board)** — group that reviews and approves normal changes.
- **Change window / maintenance window** — agreed time to make changes (often off-hours).
- **Risk & impact assessment** — what could go wrong, who's affected.
- **Implementation plan + Rollback (back-out) plan** — required for approval.
- **Freeze period** — times when changes are blocked (e.g. end-of-quarter, holidays).

### The lifecycle
```
Request (RFC) → Assess risk → Approve (CAB) → Schedule → Implement
        → Validate → Review/Close
```

> **Interview-ready:** "Change management controls production changes. Standard changes are
> pre-approved; normal changes go through the CAB; emergency changes use an expedited path.
> Every change needs an implementation plan, a change window, and a rollback plan."

---

## 5. SLA tracking

**SLA (Service Level Agreement)** = a formal commitment on service levels, with **measurable
targets** and often penalties for breaches.

### Related terms (don't mix them up)
| Term | Meaning |
|---|---|
| **SLA** | External commitment to the customer (e.g. "P1 resolved in 4 hours") |
| **OLA** | Internal agreement *between teams* supporting the SLA |
| **KPI** | A metric you measure (MTTR, uptime %) |

### Common SLA metrics
- **Response time** — how fast you acknowledge/begin work after a ticket is raised.
- **Resolution time** — how fast it's fixed (varies by priority).
- **Availability/Uptime** — e.g. "99.9%" (the famous "three nines" ≈ 8.7h downtime/year).
- **SLA compliance %** — % of tickets resolved within target (your resume cites 99%+).

### Uptime quick reference
| SLA | Downtime per year |
|---|---|
| 99% ("two nines") | ~3.65 days |
| 99.9% ("three nines") | ~8.77 hours |
| 99.99% ("four nines") | ~52.6 minutes |

- **SLA breach** — target missed; usually triggers escalation and review.
- **Pausing/clock-stop** — SLA timers can pause when waiting on the customer (status =
  "Pending"), so reporting reflects only the team's controllable time.

> **Interview-ready:** "An SLA is a measurable commitment — typically response and resolution
> times tied to priority, plus uptime. OLAs are the internal agreements that support it. We
> track SLA compliance % and act on breaches via escalation and review."

---

## 6. Monitoring (the feedback loop)

You can't meet SLAs without knowing system health. Monitoring tools watch metrics, logs,
and traces and **alert** when thresholds are crossed.
- **Metrics** — numeric measurements over time (CPU %, request latency, error rate).
- **Logs** — event records (great for diagnosis).
- **Alerts** — automated notifications when something is wrong (feed into incidents).
- **Dashboards** — visual health overview (deployment frequency, build success, etc.).
- Common tools: Azure Monitor, AWS CloudWatch, Prometheus + Grafana, Datadog, Splunk.

> **The four "golden signals"** to monitor: **Latency, Traffic, Errors, Saturation**.

---

## Practice checklist
- [ ] Explain Incident vs Problem vs Change in one sentence each.
- [ ] Build the Impact × Urgency priority matrix from memory.
- [ ] Describe the change types (standard/normal/emergency) and the CAB's role.
- [ ] State what 99.9% uptime means in downtime per year.
- [ ] Name the four golden signals of monitoring.
