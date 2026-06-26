# 01 — CI/CD & Automation

Covers: CI/CD concepts · YAML pipelines · GitHub Actions · GitHub Copilot

---

## 1. CI/CD fundamentals

### What is CI (Continuous Integration)?
Developers merge code into a shared branch **frequently** (multiple times a day).
Every merge automatically triggers a **build + automated tests**, so integration
problems are caught early instead of piling up.

- **Without CI:** developers work in isolation for weeks → painful "merge hell".
- **With CI:** small, frequent merges → small, easy-to-fix conflicts and bugs.

### What is CD?
Two related ideas:
- **Continuous Delivery** — every change that passes tests is *ready* to deploy,
  but a human clicks "approve" to push to production.
- **Continuous Deployment** — every change that passes tests goes to production
  **automatically**, no human gate.

### The typical pipeline stages
```
Source → Build → Test → Package → Deploy (Dev) → Deploy (UAT) → Deploy (Prod)
```
Each stage can **fail fast** and stop the pipeline, protecting later environments.

> **Interview-ready:** "CI is about integrating and testing code frequently; CD is
> about automating the release of that tested code. Together they reduce deployment
> risk and feedback time."

---

## 2. YAML — the language of pipelines

Almost all modern pipelines (GitHub Actions, Azure DevOps, GitLab) are defined in
**YAML** files stored *in the repo* (pipeline-as-code).

### YAML basics you must know
```yaml
# Key-value pair
name: my-pipeline

# Nesting uses INDENTATION (spaces, never tabs)
job:
  name: build
  os: ubuntu-latest

# Lists use a dash
steps:
  - checkout
  - build
  - test

# Multi-line string
script: |
  echo "line 1"
  echo "line 2"
```

Rules that trip people up:
- **Indentation is significant** — use spaces, be consistent (2 spaces is common).
- `:` separates key and value; needs a space after it.
- Strings usually don't need quotes, but use them for special chars (`:`, `#`, `*`).
- `|` keeps newlines; `>` folds lines into one.

> **Interview-ready:** "Pipeline-as-code means the build/deploy definition lives in
> version control alongside the app, so it's reviewable, auditable, and reproducible."

---

## 3. GitHub Actions (your primary CI/CD tool)

### Core concepts
| Term | What it is |
|---|---|
| **Workflow** | An automated process defined in a `.yml` file under `.github/workflows/` |
| **Event / Trigger** | What starts the workflow (`push`, `pull_request`, `schedule`, `workflow_dispatch`) |
| **Job** | A set of steps that run on the same runner |
| **Step** | A single task — either a shell command (`run`) or an action (`uses`) |
| **Action** | A reusable unit of code (e.g. `actions/checkout`) |
| **Runner** | The machine that executes the job (GitHub-hosted or self-hosted) |

### A complete example workflow
```yaml
name: CI Pipeline

# WHEN to run
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:        # allows manual trigger from UI

jobs:
  build-and-test:
    runs-on: ubuntu-latest   # the runner
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

### Deployment job with dependencies & environments
```yaml
  deploy:
    needs: build-and-test        # only runs if build-and-test succeeds
    runs-on: ubuntu-latest
    environment: production      # enables approval gates + env secrets
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}
```

### Key features to understand
- **Secrets** — store sensitive values in repo/org settings, reference as
  `${{ secrets.NAME }}`. Never hard-code credentials.
- **Environments** — named targets (Dev/UAT/Prod) that can require **approvals**
  and hold environment-specific secrets. This is how you build deployment gates.
- **`needs`** — defines job order/dependencies (creates a DAG).
- **Matrix builds** — run the same job across many versions/OSes:
  ```yaml
  strategy:
    matrix:
      node: [18, 20, 22]
  ```
- **Conditionals** — `if: github.ref == 'refs/heads/main'`
- **Reusable workflows** — call one workflow from another (`uses: ./.github/workflows/reuse.yml`)
  to avoid copy-paste. ("Reusable automation" on your resume.)
- **Caching** — `actions/cache` speeds builds by reusing dependencies.

### Common triggers cheat-sheet
| Trigger | Fires when |
|---|---|
| `push` | Code pushed to a branch |
| `pull_request` | PR opened/updated |
| `schedule` | Cron timer (`cron: '0 2 * * *'`) |
| `workflow_dispatch` | Manual button in UI |
| `release` | A GitHub release is published |

> **Interview-ready:** "A workflow is triggered by an event, contains jobs that run on
> runners, and each job has steps. I use `needs` for ordering, environments for approval
> gates, and secrets for credentials."

---

## 4. GitHub Copilot

**What it is:** an AI pair-programmer that suggests code, tests, and even whole
functions as you type, based on context.

**Where it helps a DevOps engineer:**
- Drafting pipeline YAML, Dockerfiles, and shell/PowerShell scripts.
- Writing boilerplate, regex, and `jq`/`grep` one-liners faster.
- Explaining unfamiliar code or error messages (Copilot Chat).
- Generating unit tests and documentation.

**Good habits:**
- **Always review** suggestions — Copilot can produce plausible-but-wrong or
  insecure code. You are responsible for what you commit.
- Never accept suggestions that include secrets or untrusted dependencies blindly.
- Use clear comments/function names — better context = better suggestions.

> **Interview-ready:** "Copilot accelerates writing scripts and pipeline config, but I
> treat its output as a draft to review, not as authoritative code."

---

## Practice checklist
- [ ] Create a `.github/workflows/ci.yml` that builds and tests on every push.
- [ ] Add a `deploy` job gated by `needs` and a `production` environment.
- [ ] Store a secret and reference it in a step.
- [ ] Convert a copy-pasted job into a reusable workflow.
- [ ] Add a matrix to test against 3 versions.
