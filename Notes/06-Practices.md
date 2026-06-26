# 06 — Practices

Covers: Production support (L2/L3) · Root Cause Analysis (RCA) · Post-release validation

These are the **operational disciplines** that keep production healthy. They tie together
[[04-Monitoring-and-ITSM]] (process) and [[03-Infrastructure-and-Config]] (release).

---

## 1. Production support (L2 / L3)

**What it is:** keeping live systems running, responding to issues users hit in
production, within agreed SLAs.

### The support tiers
| Tier | Scope | Typical work |
|---|---|---|
| **L1** | First line / help desk | Triage, log tickets, apply known fixes, route |
| **L2** | Technical support (your level) | Deeper diagnosis, config/data fixes, run known runbooks, coordinate |
| **L3** | Expert / engineering | Code-level debugging, infra deep-dives, permanent fixes |

- **Escalation path:** L1 → L2 → L3 → vendor/engineering. Escalate when the issue exceeds
  your scope or the SLA clock is at risk.
- **On-call rotation** — engineers take turns being available for after-hours incidents.
- **Runbooks** — step-by-step guides for known issues; the heart of fast, consistent
  support. Building a runbook/script library is what **reduces MTTR**.

### A good support workflow
```
Alert/ticket → Acknowledge (start SLA) → Triage & prioritise (Impact×Urgency)
   → Diagnose (logs, metrics, recent changes) → Apply fix or workaround
   → Validate service restored → Communicate & document → Close
   → (If recurring) raise a Problem record for RCA
```

### Diagnosis instinct — "what changed?"
Most production incidents follow a **recent change** (deploy, config, infra, data).
First questions during an incident:
1. What exactly is failing, and for whom? (scope/impact)
2. When did it start? What changed around then? (deploys, releases, config)
3. What do the **logs/metrics** say? (`tail -f`, dashboards, error rates)
4. Is there a known runbook / past incident?
5. Can I apply a **workaround** to restore service now, and fix the root cause later?

> **Interview-ready:** "L2 support is hands-on diagnosis and known-fix application within
> SLA; L3 is deep/code-level. My first move on an incident is 'what changed?' — correlate
> the start time with recent deploys and check logs. Runbooks and reusable scripts cut MTTR."

---

## 2. Root Cause Analysis (RCA)

**What it is:** the structured investigation to find the *underlying cause* of an incident
so it **doesn't happen again**. (This is ITIL **Problem Management**.)

> Incident management restores service *now*; RCA prevents the *next* occurrence.

### The 5 Whys technique
Keep asking "why" until you reach the true cause:
```
Problem: The website went down.
1. Why? The server ran out of memory.
2. Why? A process leaked memory.
3. Why? A recent code change didn't release database connections.
4. Why? The connection-pool wasn't closed in error paths.
5. Why? No code review check / test covered that path.   ← root cause
```
Fix the *root* (add review/test + close connections), not just the symptom (restart server).

### Other RCA tools
- **Fishbone / Ishikawa diagram** — categorise possible causes (People, Process,
  Technology, Environment).
- **Timeline analysis** — reconstruct exactly what happened and when.
- **Pareto principle (80/20)** — a few causes drive most incidents; tackle those first.

### A good RCA / post-mortem document contains
1. **Summary** — what happened, impact, duration.
2. **Timeline** — detection → diagnosis → resolution.
3. **Root cause** — the actual underlying cause (not just symptoms).
4. **Resolution** — what fixed it.
5. **Corrective / preventive actions** — concrete tasks with owners and dates.
6. **Lessons learned.**

### Blameless culture
Modern post-mortems are **blameless** — focus on *systems and processes that allowed* the
error, not on punishing individuals. This encourages honesty and real fixes.

> **Interview-ready:** "RCA finds why an incident happened so we prevent recurrence. I use
> the 5 Whys to get past symptoms to the root cause, then document corrective actions with
> owners. We keep post-mortems blameless."

---

## 3. Post-release validation

**What it is:** the checks performed **right after a deployment** to confirm the release is
healthy and the system works as expected in production.

### Before release — pre-deployment checklist
- [ ] Change approved & in the change window.
- [ ] Backups / database snapshot taken.
- [ ] **Rollback plan** documented and tested.
- [ ] Stakeholders notified.
- [ ] Monitoring/alerting in place.

### After release — validation steps
| Check | What it confirms |
|---|---|
| **Smoke tests** | Critical paths work (login, key transaction, homepage loads) |
| **Health checks** | App `/health` endpoints return OK; services up |
| **Sanity checks** | Version deployed is the expected one; config correct |
| **Monitoring review** | Error rates, latency, CPU/memory are normal (no spikes) |
| **Log review** | No new errors/exceptions after deploy |
| **Business validation** | QA / business confirm key features (esp. in UAT) |

These can (and should) be **automated in the pipeline** — e.g. a GitHub Actions step that
hits a health endpoint and fails the deploy if it's not 200. (Automating these is what
"reduced deployment time / validation steps" on your resume refers to.)

```yaml
# Example post-deploy smoke test step in GitHub Actions
- name: Smoke test
  run: |
    status=$(curl -s -o /dev/null -w "%{http_code}" https://app.example.com/health)
    if [ "$status" -ne 200 ]; then
      echo "Health check failed with $status"
      exit 1     # fail the pipeline → trigger rollback / alert
    fi
```

### If validation fails
Execute the **rollback plan** (blue-green switch back, redeploy previous version, restore
backup), communicate, then RCA the failure.

> **Interview-ready:** "Post-release validation is the smoke/health/sanity checks right
> after deploy to confirm production is healthy. I automate them in the pipeline so a failed
> health check fails the deploy, and I always have a rollback plan ready if validation fails."

---

## How it all connects (the DevOps loop)

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (Plan)
 │                     │        │         │         │         │
 │              GitHub Actions  │   Release mgmt   L2/L3    ServiceNow
 │              YAML pipelines  │   + cutover     support   + SLA tracking
 └── branching strategy    Post-release validation    RCA feeds back into Plan
```

- A **monitor** alert → becomes an **incident** (ServiceNow) → worked by **L2/L3 support**
  within **SLA** → if recurring, **RCA** finds the root cause → the fix becomes a **change**
  → deployed via **CI/CD pipeline** → confirmed by **post-release validation**.
- Everything is **automated** where possible (scripts, pipelines) and **controlled** where
  it matters (approvals, change management).

---

## Practice checklist
- [ ] Walk through an incident from alert to closure out loud.
- [ ] Do a 5 Whys on a made-up outage down to a root cause.
- [ ] Write a pre-deployment checklist and a post-release smoke-test list.
- [ ] Add an automated health-check step to a pipeline that fails on non-200.
- [ ] Explain the difference between incident resolution and RCA.
