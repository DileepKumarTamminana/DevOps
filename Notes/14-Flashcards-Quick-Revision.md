# 14 — Flashcards / Quick Revision

Active recall beats re-reading. Cover the **A:** line, answer out loud, then check. Cards are
grouped by topic and cross-link to the full notes. Do a few groups a day; star the ones you miss.

> How to use: read **Q**, say your answer aloud, reveal **A**. If wrong/slow → revisit the
> linked note. Aim to get a whole group right two days running.

---

## CI/CD & Automation → [[01-CICD-and-Automation]]
- **Q:** CI vs CD in one line each?
  **A:** CI = integrate & auto-test code frequently. CD = automate releasing that tested code (Delivery = gated; Deployment = automatic).
- **Q:** Workflow → ? → ? → ? (the hierarchy)
  **A:** Workflow → Jobs → Steps; jobs run on Runners; steps are `run` (command) or `uses` (action).
- **Q:** How do you order jobs and gate a prod deploy in GitHub Actions?
  **A:** `needs:` for ordering; `environment:` with required approvals for the gate.
- **Q:** Where do credentials go, and how are they referenced?
  **A:** In secret stores; referenced as `${{ secrets.NAME }}` — never hard-coded.
- **Q:** What's a matrix build?
  **A:** Run the same job across multiple versions/OSes in parallel.

## Git → [[11-Git-Essentials]]
- **Q:** The three Git areas?
  **A:** Working directory → staging (index) → local repo (then push to remote).
- **Q:** `merge` vs `rebase`?
  **A:** Merge preserves history with a merge commit; rebase rewrites it linearly. Never rebase shared/pushed commits.
- **Q:** Undo a commit that's already pushed?
  **A:** `git revert` — adds a new commit, safe on shared history. (`reset --hard` is local-only.)
- **Q:** `git fetch` vs `git pull`?
  **A:** `pull` = `fetch` + `merge`. `fetch` downloads without merging.
- **Q:** What belongs in `.gitignore`?
  **A:** Build output, dependencies (`node_modules/`), and secrets (`.env`) — never commit credentials.

## Cloud → [[02-Cloud-Platforms]]
- **Q:** IaaS vs PaaS vs SaaS?
  **A:** IaaS = you manage OS+app (VM); PaaS = deploy code, provider runs OS (App Service); SaaS = ready software (O365).
- **Q:** Region vs Availability Zone?
  **A:** Region = geographic area; AZ = physically isolated datacenter within a region (for HA).
- **Q:** What is a resource group? RBAC?
  **A:** RG = logical container managed together. RBAC = roles (Owner/Contributor/Reader) at a scope; least privilege.
- **Q:** Map EC2, S3, IAM, Lambda to Azure.
  **A:** VM, Blob Storage, Entra ID + RBAC, Functions.
- **Q:** CapEx → OpEx means?
  **A:** Cloud turns big upfront hardware spend into pay-as-you-go operating cost.

## Docker → [[03-Infrastructure-and-Config]]
- **Q:** Image vs container?
  **A:** Image = read-only template; container = running instance of it.
- **Q:** Container vs VM?
  **A:** Container shares the host kernel (MBs, seconds); VM bundles a full OS (GBs, minutes).
- **Q:** Why copy `package.json` before the source in a Dockerfile?
  **A:** Layer caching — code changes won't bust the cached dependency-install layer.
- **Q:** Where do secrets go in containers?
  **A:** Injected at runtime (env vars / secret store) — never baked into the image.

## Linux & Scripting → [[03-Infrastructure-and-Config]], [[05-Scripting-and-Query]]
- **Q:** `chmod 755` means?
  **A:** Owner rwx(7), group r-x(5), others r-x(5). r=4, w=2, x=1.
- **Q:** Follow a live log? A service's logs?
  **A:** `tail -f file` / `journalctl -u <svc> -f`.
- **Q:** What does `set -euo pipefail` do?
  **A:** Exit on error, error on unset vars, fail on broken pipes — safe scripts.
- **Q:** Bash vs PowerShell key difference?
  **A:** Bash pipes text; PowerShell pipes objects (filter on properties with `Where-Object`).
- **Q:** Disk is full — first three commands?
  **A:** `df -h` (confirm) → `du -sh /*` (find big dirs) → check/rotate `/var/log`.

## SQL → [[05-Scripting-and-Query]]
- **Q:** WHERE vs HAVING?
  **A:** WHERE filters rows before grouping; HAVING filters after aggregation.
- **Q:** INNER vs LEFT JOIN?
  **A:** INNER = only matching rows in both; LEFT = all left rows + matches (NULLs if none).
- **Q:** Logical execution order of a SELECT?
  **A:** FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT.
- **Q:** Golden rule for UPDATE/DELETE?
  **A:** Always include a WHERE — without it you change every row.

## Monitoring & ITSM → [[04-Monitoring-and-ITSM]]
- **Q:** Incident vs Problem vs Change?
  **A:** Incident = unplanned interruption (restore fast); Problem = root cause of incidents; Change = planned modification.
- **Q:** How is priority decided?
  **A:** Priority = Impact × Urgency → P1–P4, which sets the SLA.
- **Q:** Standard vs Normal vs Emergency change?
  **A:** Standard = pre-approved; Normal = CAB-approved; Emergency = expedited (ECAB).
- **Q:** SLA vs OLA vs KPI?
  **A:** SLA = external commitment; OLA = internal team agreement; KPI = a measured metric.
- **Q:** 99.9% uptime = how much downtime/year?
  **A:** ~8.77 hours ("three nines").
- **Q:** The four golden signals?
  **A:** Latency, Traffic, Errors, Saturation.

## Practices → [[06-Practices]]
- **Q:** First instinct on a production incident?
  **A:** "What changed?" — correlate start time with recent deploys/config, then check logs.
- **Q:** RCA technique and its goal?
  **A:** 5 Whys; goal is to prevent recurrence, not just restore service.
- **Q:** What is post-release validation?
  **A:** Smoke/health/sanity checks after deploy to confirm production is healthy; automate to fail the deploy on a bad check.
- **Q:** Blue-green vs canary?
  **A:** Blue-green = swap two full environments (instant rollback); canary = release to a small % first.
- **Q:** What's always required for a prod release/change?
  **A:** A tested rollback plan.

## Kubernetes → [[12-Kubernetes-Basics]]
- **Q:** What problem does K8s solve?
  **A:** Orchestrates many containers across machines — self-healing, scaling, rolling updates, load balancing.
- **Q:** Pod vs Deployment vs Service?
  **A:** Pod = smallest unit (runs containers); Deployment = manages N Pods + rolling updates/rollback; Service = stable endpoint + load balancing.
- **Q:** How does a Service know which Pods to route to?
  **A:** Label/selector matching.
- **Q:** `CrashLoopBackOff` — how to debug?
  **A:** `kubectl describe pod` (events) + `kubectl logs` (app errors).
- **Q:** Liveness vs readiness probe?
  **A:** Liveness = restart if unhealthy; readiness = don't send traffic until ready.

---

## 60-second "can I explain DevOps?" challenge
Say this out loud without notes:
> "DevOps shortens the gap between writing code and running it reliably. Code goes through a
> CI pipeline that builds and tests on every push, produces an immutable artifact, and a CD
> stage deploys it — gated by approvals for production. Infrastructure is code, releases use
> blue-green/canary with health checks and a rollback plan, and monitoring feeds incidents
> into ITSM where we work them within SLA and do RCA to prevent recurrence."

If you can deliver that fluently, you can frame any DevOps interview answer.

---

## Spaced-repetition plan
- **Pass 1 (today):** read every card, mark misses with a ⭐.
- **Pass 2 (tomorrow):** only the ⭐ cards.
- **Pass 3 (+3 days):** all cards again; you should clear most.
- **Pass 4 (+1 week):** final sweep before the interview.
