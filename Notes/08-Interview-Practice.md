# 08 — Interview Practice

Q&A organised by topic, plus scenario questions, your-resume questions, and a rapid-fire
round. Practice **answering out loud** — knowing isn't the same as explaining.

> Format tip for answers: **Define → How it works → Example/why it matters.** For scenarios,
> use **STAR** (Situation, Task, Action, Result).

---

## A. CI/CD & Automation

**Q1. What is the difference between Continuous Delivery and Continuous Deployment?**
Both automate everything up to production. **Delivery** stops at a manual approval gate —
a human decides when to release. **Deployment** goes all the way to production
automatically with no human gate. Delivery = "ready to ship anytime"; Deployment = "ships
on every green build."

**Q2. Explain the structure of a GitHub Actions workflow.**
A **workflow** (YAML in `.github/workflows/`) is triggered by an **event** (push, PR,
schedule, manual). It contains **jobs** that run on **runners**; each job has **steps**
that either run shell commands (`run`) or reusable **actions** (`uses`). `needs` defines
job order; `environments` add approval gates; `secrets` hold credentials.

**Q3. How do you handle secrets in a pipeline?**
Never hard-code them. Store in GitHub/Azure DevOps secret stores (or a vault like Azure Key
Vault / AWS Secrets Manager) and reference at runtime (`${{ secrets.NAME }}`). Scope them
per-environment and rotate regularly.

**Q4. What's a matrix build and why use it?**
Running the same job across combinations (e.g. Node 18/20/22, Linux/Windows) in parallel.
It catches version/OS-specific issues without duplicating workflow code.

**Q5. How do you avoid duplicating pipeline code?**
Reusable workflows and composite actions — define once, call from many workflows. Keeps
pipelines DRY and consistent.

---

## B. Cloud (Azure / AWS)

**Q6. IaaS vs PaaS vs SaaS?**
IaaS = you manage OS + app (VM/EC2). PaaS = provider manages OS/runtime, you deploy code
(App Service/Beanstalk). SaaS = ready-to-use software (Office 365, ServiceNow).

**Q7. What is a resource group in Azure?**
A logical container for related resources that you manage, deploy, and delete together;
also a scope for RBAC and cost tracking.

**Q8. How does RBAC work?**
Role-Based Access Control assigns **roles** (Owner/Contributor/Reader) to **identities**
at a **scope** (subscription / resource group / resource). Follow least privilege — grant
the minimum needed.

**Q9. Region vs Availability Zone?**
A region is a geographic area of datacenters; availability zones are physically separate
datacenters within a region, used for high availability and fault tolerance.

**Q10. Map AWS to Azure: EC2, S3, IAM, Lambda.**
EC2→VM, S3→Blob Storage, IAM→Entra ID + RBAC, Lambda→Azure Functions.

**Q11. How do you control cloud cost?**
Budgets + alerts (Azure Cost Management / AWS Budgets), right-sizing, shutting down idle
resources, reserved instances for steady workloads, and tagging for cost attribution.

---

## C. Docker & Linux

**Q12. Container vs VM?**
A VM bundles a full guest OS (GBs, slow boot); a container packages just the app +
dependencies and shares the host kernel (MBs, seconds to start). Containers are lighter
and more portable.

**Q13. Image vs container?**
An image is a read-only template; a container is a running instance of an image. One image
→ many containers.

**Q14. What is Docker layer caching and why does order matter?**
Each Dockerfile instruction is a cached layer. Copy dependency manifests and install
*before* copying source code, so a code change doesn't invalidate the (expensive)
dependency-install layer.

**Q15. Explain `chmod 755`.**
Owner = rwx (7), group = r-x (5), others = r-x (5). r=4, w=2, x=1.

**Q16. A Linux server's disk is full — how do you investigate?**
`df -h` to confirm, `du -sh /*` (then drill down) to find the big directories, check
`/var/log` for runaway logs, clear/rotate or compress, and set up log rotation to prevent
recurrence.

**Q17. How do you view live logs and a service's status?**
`tail -f app.log` or `journalctl -u <service> -f`; `systemctl status <service>` for state,
`systemctl restart <service>` to restart.

---

## D. Scripting & SQL

**Q18. What does `set -euo pipefail` do?**
`-e` exit on any error, `-u` error on unset variables, `-o pipefail` fail if any command in
a pipe fails. It makes scripts fail fast and safe.

**Q19. What's the difference between WHERE and HAVING?**
`WHERE` filters rows before grouping; `HAVING` filters after aggregation (on `GROUP BY`
results). You can't use aggregate functions in `WHERE`.

**Q20. INNER JOIN vs LEFT JOIN?**
INNER returns only rows that match in both tables; LEFT returns all rows from the left
table plus matches from the right (NULLs where no match).

**Q21. Write SQL: count employees per department, only departments with more than 5.**
```sql
SELECT department, COUNT(*) AS cnt
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```

**Q22. Bash vs PowerShell — key difference?**
Bash pipes plain **text**; PowerShell pipes **objects**, so you filter on properties
(`Where-Object { $_.CPU -gt 100 }`) without text parsing.

---

## E. ITSM / Support (ServiceNow, SLA, Incident, Change)

**Q23. Incident vs Problem vs Change?**
Incident = unplanned interruption (restore service fast). Problem = the underlying root
cause of incidents (prevent recurrence). Change = a planned modification to the
environment (done in a controlled way).

**Q24. How is incident priority decided?**
Priority = **Impact × Urgency**. Impact = how many users/how much business is affected;
urgency = how fast it must be fixed. This sets the SLA and escalation path.

**Q25. What's the difference between L1, L2, and L3 support?**
L1 = first response/triage and known fixes; L2 = deeper technical troubleshooting; L3 =
specialists/engineering for code- or infra-level fixes.

**Q26. What is a CAB?**
Change Advisory Board — the group that reviews and approves normal changes, assessing risk,
impact, and rollback plans before they're scheduled.

**Q27. Standard vs Normal vs Emergency change?**
Standard = low-risk, pre-approved, repeatable. Normal = needs assessment + CAB approval.
Emergency = urgent fix for a major incident via an expedited (ECAB) path.

**Q28. What does 99.9% uptime mean?**
About 8.77 hours of allowed downtime per year ("three nines").

**Q29. SLA vs OLA vs KPI?**
SLA = external commitment to the customer; OLA = internal agreement between teams that
supports the SLA; KPI = a metric you measure (MTTR, uptime%).

**Q30. What is MTTR and how do you improve it?**
Mean Time To Resolve/Recover. Improve it with runbooks, reusable diagnostic/automation
scripts, good monitoring/alerting, and clear escalation paths.

---

## F. Practices (RCA, Validation, Release)

**Q31. Walk me through handling a P1 production incident.**
Acknowledge (start SLA) → assess impact/scope → check what changed recently + logs/metrics
→ apply a workaround to restore service → validate service is back → communicate to
stakeholders → document → if recurring, open a Problem record for RCA.

**Q32. What is RCA and what technique do you use?**
Root Cause Analysis finds the underlying cause so an incident doesn't recur. I use the
**5 Whys** to move past symptoms to the real cause, then document corrective actions with
owners. Post-mortems are blameless.

**Q33. What is post-release validation?**
Checks right after deploy — smoke tests (critical paths), health endpoints, version/config
sanity, and monitoring/log review — to confirm production is healthy. I automate them in
the pipeline so a failed check fails the deploy.

**Q34. Explain blue-green and canary deployments.**
Blue-green: two identical environments; switch traffic to the new one, enabling instant
rollback. Canary: release to a small percentage of users first to limit blast radius
before full rollout.

**Q35. What goes in a rollback plan?**
The exact steps to revert (traffic switch / redeploy previous version / restore backup),
who executes it, the trigger conditions, and how you verify the rollback succeeded.

**Q36. What is a cutover?**
The planned switch from an old system/version to a new one, often during migration. It
needs a runbook, timeline, defined roles, a go/no-go checkpoint, and a rollback plan —
aiming for zero downtime.

---

## G. Scenario / behavioural (use STAR)

**S1.** *A deployment to production failed and the site is down. What do you do?*
Restore service first (rollback to last good version / blue-green switch), confirm health,
communicate status, then RCA the failure — don't debug live in prod while users are down.

**S2.** *A pipeline that was green yesterday now fails intermittently. How do you debug?*
Identify if it's a flaky test, a timing/race issue, an external dependency, or
infra/runner variance. Re-run to confirm flakiness, check recent changes, add logging,
isolate the failing stage, and fix the root cause rather than just retrying.

**S3.** *Two teams want to deploy during the same change window. How do you handle it?*
Check dependencies and risk, coordinate via change management/CAB, sequence the changes,
ensure each has its own rollback plan, and communicate the agreed schedule.

**S4.** *You notice the same incident recurring weekly. What do you do?*
Raise a Problem record, perform RCA (5 Whys), implement a permanent fix or automation, and
add monitoring so it's caught earlier — converting reactive firefighting into prevention.

**S5.** *How do you ensure a zero-downtime release during a migration?*
Pre-deployment checklist, backups, rehearsed cutover runbook, blue-green or rolling
strategy, post-release smoke tests, and a tested rollback plan with a go/no-go gate.

---

## H. Questions about YOUR resume (be ready to defend every line)

- "You claim 99%+ SLA compliance — how was it measured and what did *you* do to maintain it?"
- "Describe a specific RCA you led — the incident, the root cause, and the fix."
- "Walk me through a CI/CD pipeline you built in GitHub Actions, stage by stage."
- "You led cutover activities — what was your runbook and rollback plan?"
- "Give an example of a reusable automation script and the manual effort it removed."
- "How did you achieve zero-downtime releases? Which deployment strategy?"
- "Tell me about a time a deployment went wrong and how you recovered."
> For each, prepare a concrete STAR story with **numbers** (time saved, MTTR reduced, % SLA).

---

## I. Rapid-fire (one-line answers)
- Port for HTTPS? **443.** HTTP? **80.** SSH? **22.**
- What does `git rebase` do? **Replays commits onto a new base (linear history).**
- `git merge` vs `git rebase`? **Merge preserves history with a merge commit; rebase rewrites it linearly.**
- What is a `.gitignore`? **Lists files Git should not track.**
- Exit code 0 means? **Success.**
- `docker exec` vs `docker run`? **exec runs a command in an existing container; run starts a new one.**
- What is idempotency? **An operation that gives the same result no matter how many times it runs.**
- Stateless vs stateful? **Stateless keeps no client data between requests; stateful does.**
- What is a webhook? **An HTTP callback that fires on an event (e.g. push triggers a pipeline).**
- What is the CMDB? **The database of configuration items (assets) and their relationships.**

---

## How to prepare (1-week plan)
- **Day 1–2:** Re-read [[01-CICD-and-Automation]] + [[03-Infrastructure-and-Config]]; answer sections A & C aloud.
- **Day 3:** Cloud — [[02-Cloud-Platforms]]; sections B.
- **Day 4:** Scripting/SQL — [[05-Scripting-and-Query]]; section D + write the SQL/Bash by hand.
- **Day 5:** ITSM + Practices — [[04-Monitoring-and-ITSM]], [[06-Practices]]; sections E & F.
- **Day 6:** Scenarios (G) + resume stories (H) — write STAR answers with numbers.
- **Day 7:** Rapid-fire (I) + do one hands-on project from [[07-HandsOn-Projects]] to have a demo.
