# 10 — Complete Project: "Welcome Text" via a full AWS pipeline

One end-to-end project that ties together **everything**: an app → Git → a CI/CD pipeline
(GitHub Actions) → a container image in **Amazon ECR** → a running service on **AWS App
Runner** → a live URL showing **"Welcome Text"** → an automated post-deploy smoke test.

This is the project to put on your resume — it proves CI, CD, Docker, AWS, secrets, and
post-release validation in a single link.

> Written for **Windows + PowerShell**. `PS>` = type in PowerShell. "✅ Expect" = success.
> "❌ If you see…" = the fix. Replace `<…>` placeholders with your values.

---

## What you'll build (architecture)

```
   You push code to GitHub
            │
            ▼
   ┌─────────────────────────────────────────────┐
   │            GitHub Actions pipeline           │
   │                                              │
   │  1. Checkout   2. Test   3. Build image      │
   │  4. Push image → Amazon ECR                  │
   │  5. Deploy → AWS App Runner                  │
   │  6. Smoke test the live URL                  │
   └─────────────────────────────────────────────┘
            │                         │
            ▼                         ▼
     Amazon ECR (registry)     AWS App Runner (runs the container)
                                       │
                                       ▼
                         https://xxxx.awsapprunner.com  → "Welcome Text"
```

**Why these AWS services:**
- **ECR (Elastic Container Registry)** — AWS's private Docker image store (≈ Docker Hub).
- **App Runner** — the simplest way to run a container as a web service on AWS; it gives you
  an HTTPS URL automatically, no servers or networking to manage.

> 💡 **Cost note:** App Runner is **not** fully free-tier — it bills while running (~a few
> cents/hour). Do the **Cleanup** step at the end to stop charges. A zero-cost alternative
> (S3 static site) is in the appendix.

---

## Prerequisites

| Need | Get it / verify |
|---|---|
| AWS account | https://aws.amazon.com/free |
| AWS CLI v2 | https://aws.amazon.com/cli → `aws --version` |
| Docker Desktop running | `docker --version` + whale icon "Engine running" |
| Git + GitHub account | `git --version` |
| Node.js | `node --version` |

Configure the CLI once with an admin/your IAM user:
```powershell
aws configure
# enter Access Key, Secret Key, default region (use: us-east-1), output: json
aws sts get-caller-identity      # ✅ Expect: your account number + user ARN
```

Pick a region and remember it (this guide uses **us-east-1**).

---

## Part 1 — Create the application

**1.1** Make the project folder and open VS Code:
```powershell
cd "$HOME\Documents\devops-projects"
mkdir welcome-aws; cd welcome-aws
code .
```

**1.2** Create `server.js` — serves the welcome text and a health endpoint:
```javascript
const http = require('http');

const PORT = process.env.PORT || 8080;   // App Runner sets PORT

const server = http.createServer((req, res) => {
  if (req.url === '/health') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    return res.end('OK');
  }
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end(`
    <html>
      <head><title>Welcome</title></head>
      <body style="font-family: sans-serif; text-align: center; margin-top: 80px;">
        <h1>👋 Welcome Text</h1>
        <p>Deployed to AWS App Runner via a GitHub Actions CI/CD pipeline.</p>
        <p>Build: ${process.env.BUILD_SHA || 'local'}</p>
      </body>
    </html>
  `);
});

server.listen(PORT, () => console.log(`Listening on ${PORT}`));
```

**1.3** Create `Dockerfile`:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY server.js .
EXPOSE 8080
CMD ["node", "server.js"]
```

**1.4** Test it locally before involving AWS:
```powershell
docker build -t welcome-aws:local .
docker run -d -p 8080:8080 --name welcome welcome-aws:local
```
Open http://localhost:8080 → ✅ Expect the **"👋 Welcome Text"** page. Then clean up local:
```powershell
docker stop welcome; docker rm welcome
```

---

## Part 2 — Create the AWS resources (one-time)

**2.1 Create the ECR repository** (where images will live):
```powershell
aws ecr create-repository --repository-name welcome-aws --region us-east-1
```
✅ Expect: JSON with a `repositoryUri` like
`<acct-id>.dkr.ecr.us-east-1.amazonaws.com/welcome-aws`. **Copy your account ID** (the
12-digit number) — you'll need it.

**2.2 Push a first image manually** so App Runner has something to start from:
```powershell
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <acct-id>.dkr.ecr.us-east-1.amazonaws.com

# Tag and push the local image
docker tag welcome-aws:local <acct-id>.dkr.ecr.us-east-1.amazonaws.com/welcome-aws:latest
docker push <acct-id>.dkr.ecr.us-east-1.amazonaws.com/welcome-aws:latest
```
✅ Expect: layers push and a digest is printed.

**2.3 Create the App Runner service** (Console is easiest the first time):
1. AWS Console → search **App Runner** → **Create service**.
2. Source: **Container registry** → **Amazon ECR** → **Browse** → pick `welcome-aws:latest`.
3. Deployment trigger: **Manual** (the pipeline will trigger deploys).
4. ECR access role: let it **create a new service role** when prompted.
5. Service name: `welcome-aws`. Port: **8080**. Leave CPU/memory at smallest.
6. **Create & deploy** → wait until status is **Running** (~3–5 min).
7. Copy the **Default domain** (e.g. `https://abcd1234.us-east-1.awsapprunner.com`).

Open that URL → ✅ Expect the **Welcome Text** page, now live on AWS. 🎉

**2.4 Get the service ARN** (the pipeline needs it):
```powershell
aws apprunner list-services --region us-east-1
```
Copy the `ServiceArn` value for `welcome-aws`.

---

## Part 3 — Create an IAM user for the pipeline

The pipeline needs AWS credentials. Create a dedicated, least-privilege user.

**3.1** Console → **IAM** → **Users** → **Create user** → name `github-actions-welcome`.
**3.2** Attach policies (for learning; tighten later): `AmazonEC2ContainerRegistryFullAccess`
and `AWSAppRunnerFullAccess`.
**3.3** After creating, open the user → **Security credentials** → **Create access key** →
use case **Other** → copy the **Access key ID** and **Secret access key** (shown once).

> 🔐 **Better practice (mention in interviews):** in production use **GitHub OIDC** with an
> IAM role instead of long-lived access keys — no secrets stored at all. Keys are fine for
> learning.

---

## Part 4 — Wire up the GitHub repo & secrets

**4.1** Initialise git and push the app:
```powershell
cd "$HOME\Documents\devops-projects\welcome-aws"
git init
"node_modules/" | Out-File -Encoding utf8 .gitignore
git add .
git commit -m "Welcome app + Dockerfile"
```
Create an empty repo `welcome-aws` on GitHub (no README), then:
```powershell
git remote add origin https://github.com/<your-username>/welcome-aws.git
git branch -M main
git push -u origin main
```

**4.2** In the GitHub repo → **Settings → Secrets and variables → Actions → New repository
secret**. Add these four:

| Secret name | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | from Part 3.3 |
| `AWS_SECRET_ACCESS_KEY` | from Part 3.3 |
| `ECR_REPOSITORY` | `welcome-aws` |
| `APPRUNNER_SERVICE_ARN` | the ARN from Part 2.4 |

---

## Part 5 — The CI/CD pipeline

Create `.github/workflows/deploy.yml`:
```yaml
name: Build, Push & Deploy to AWS

on:
  push:
    branches: [ main ]
  workflow_dispatch:          # allow manual run from the Actions tab

env:
  AWS_REGION: us-east-1

jobs:
  pipeline:
    runs-on: ubuntu-latest
    steps:
      # 1. Get the code
      - name: Checkout
        uses: actions/checkout@v4

      # 2. Test stage (basic sanity check)
      - name: Test - app file present
        run: test -f server.js && echo "server.js exists ✓"

      # 3. Configure AWS credentials
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      # 4. Log in to Amazon ECR
      - name: Login to Amazon ECR
        id: ecr
        uses: aws-actions/amazon-ecr-login@v2

      # 5. Build & push the image, tagged with the commit SHA
      - name: Build and push image
        env:
          REGISTRY: ${{ steps.ecr.outputs.registry }}
          REPO: ${{ secrets.ECR_REPOSITORY }}
          TAG: ${{ github.sha }}
        run: |
          docker build -t "$REGISTRY/$REPO:$TAG" -t "$REGISTRY/$REPO:latest" .
          docker push "$REGISTRY/$REPO:$TAG"
          docker push "$REGISTRY/$REPO:latest"

      # 6. Deploy: trigger an App Runner deployment
      - name: Deploy to App Runner
        run: |
          aws apprunner start-deployment \
            --service-arn "${{ secrets.APPRUNNER_SERVICE_ARN }}" \
            --region "$AWS_REGION"

      # 7. Wait until the service is back to RUNNING
      - name: Wait for service to be RUNNING
        run: |
          for i in $(seq 1 30); do
            status=$(aws apprunner describe-service \
              --service-arn "${{ secrets.APPRUNNER_SERVICE_ARN }}" \
              --region "$AWS_REGION" --query "Service.Status" --output text)
            echo "Attempt $i: status=$status"
            [ "$status" = "RUNNING" ] && break
            sleep 20
          done

      # 8. Post-release smoke test against the live URL
      - name: Smoke test
        run: |
          URL=$(aws apprunner describe-service \
            --service-arn "${{ secrets.APPRUNNER_SERVICE_ARN }}" \
            --region "$AWS_REGION" --query "Service.ServiceUrl" --output text)
          echo "Testing https://$URL/health"
          code=$(curl -s -o /dev/null -w "%{http_code}" "https://$URL/health")
          if [ "$code" -eq 200 ]; then
            echo "✓ Live and healthy: https://$URL"
          else
            echo "✗ Health check failed with HTTP $code"; exit 1
          fi
```

---

## Part 5b — Understand the pipeline: what each action does & why

This is the most important section for interviews — **be able to explain every line**.

### Detailed architecture & data flow

```
 ┌──────────┐   git push    ┌─────────────────────────────────────────────────────────┐
 │   YOU    │ ────────────► │                  GitHub repository                       │
 │ (laptop) │   to main     │   server.js · Dockerfile · .github/workflows/deploy.yml  │
 └──────────┘               └───────────────────────────┬─────────────────────────────┘
                                                         │ push event triggers workflow
                                                         ▼
 ┌──────────────────────────── GitHub Actions runner (ubuntu-latest, fresh VM) ─────────────────────────────┐
 │                                                                                                            │
 │  [1] checkout ─► [2] test ─► [3] configure AWS creds ─► [4] ECR login ─► [5] build+push image             │
 │       │                                  │                    │                  │                         │
 │       │                                  │ uses GitHub        │ gets a temp      │ docker build → push     │
 │       │                                  │ secrets (keys)     │ ECR token        │ two tags: <sha>+latest  │
 │       ▼                                  ▼                    ▼                  ▼                         │
 │   code on VM                      AWS identity set      authenticated     ┌───────────────┐               │
 │                                                          to ECR           │  Amazon ECR    │◄── image      │
 │                                                                           │ (image store)  │               │
 │  [6] deploy ──────────────────────────────────────────────────────────► └───────┬───────┘               │
 │       │ aws apprunner start-deployment                                            │ App Runner pulls       │
 │       ▼                                                                            ▼ the new image          │
 │  [7] wait for RUNNING ◄──────────────────────────── poll status ──────── ┌────────────────┐               │
 │       │                                                                   │ AWS App Runner  │               │
 │  [8] smoke test ──► curl https://<url>/health ──────────────────────────►│ (runs container)│               │
 │       │             expects HTTP 200, else fail the build                 └───────┬────────┘               │
 └───────┼───────────────────────────────────────────────────────────────────────────┼─────────────────────┘
         │ pipeline goes green ✓                                                       │ serves on :8080
         ▼                                                                              ▼
   You see "deploy succeeded"                                    https://xxxx.awsapprunner.com → "Welcome Text"
```

### The two kinds of "action" in the YAML
- **`uses:`** — a *prebuilt, reusable action* published by someone else (referenced as
  `owner/name@version`). You're calling code maintained by GitHub or AWS.
- **`run:`** — *your own shell commands* executed on the runner.

### Step-by-step: purpose of each action

| # | YAML | Type | What it does | Why it's needed (purpose) |
|---|---|---|---|---|
| Trigger | `on: push: branches:[main]` | event | Starts the workflow when code lands on `main` | This is the **CI/CD trigger** — every merge to main auto-deploys (continuous deployment) |
| Trigger | `workflow_dispatch` | event | Adds a manual "Run workflow" button | Lets you redeploy on demand without a code change |
| Runner | `runs-on: ubuntu-latest` | infra | Spins up a fresh, clean Linux VM | Builds are **reproducible** — no leftover state from previous runs |
| 1 | `actions/checkout@v4` | uses | Clones your repo onto the runner | Without it the runner is empty — nothing to build |
| 2 | `test -f server.js` | run | Sanity check the app file exists | A **test/quality gate** — fail fast before spending time building/deploying |
| 3 | `aws-actions/configure-aws-credentials@v4` | uses | Loads AWS keys (from secrets) into the environment | **Authenticates** the runner to your AWS account; every later `aws`/ECR call needs this |
| 4 | `aws-actions/amazon-ecr-login@v2` | uses | Gets a short-lived token & runs `docker login` to ECR | Docker can't push to a **private** registry without logging in |
| 5 | `docker build … && docker push …` | run | Builds the image, tags it `:<sha>` and `:latest`, pushes both to ECR | Produces the **deployable artifact**. The `:<sha>` tag makes every build **traceable** to a commit (and rollback-able) |
| 6 | `aws apprunner start-deployment` | run | Tells App Runner to pull the new image and roll it out | This is the **deploy (CD) step** — promotes the artifact to the running service |
| 7 | `aws apprunner describe-service` loop | run | Polls service status until `RUNNING` | A deploy is async; we **wait for readiness** so the smoke test doesn't run too early |
| 8 | `curl …/health` → check 200 | run | Hits the live URL; fails the job if not HTTP 200 | **Post-release validation** — automatically proves the deploy is actually healthy, not just "finished" |

### Why each AWS piece exists (the "why this service")
| Service | Role in the pipeline | Analogy |
|---|---|---|
| **ECR** | Private store for the built image | Like Docker Hub, but inside your AWS account |
| **App Runner** | Runs the container & exposes an HTTPS URL | A managed "run my container" button — no servers/networking to manage |
| **IAM user + keys** | The pipeline's identity & permissions | A service account the robot logs in as (least privilege) |
| **GitHub Secrets** | Encrypted store for the AWS keys + ARNs | A vault so credentials never sit in the code |

### The DevOps principles this pipeline demonstrates
- **Pipeline-as-code** — the whole process lives in `deploy.yml`, versioned with the app.
- **Immutable, traceable artifacts** — image tagged by commit SHA; you can always tell what's
  running and roll back to a prior tag.
- **Separation of build vs deploy** — build/push (steps 1–5) is CI; deploy (step 6) is CD.
- **Fail fast & validate** — test gate up front (step 2), health gate at the end (step 8).
- **Secrets externalised** — no credentials in the repo (step 3 pulls from the vault).
- **Idempotent & reproducible** — a clean runner + declarative config = same result every run.

> **Interview-ready (one breath):** "The push triggers a fresh runner that checks out the
> code, runs a test gate, authenticates to AWS, logs in to ECR, builds an image tagged with
> the commit SHA, pushes it, then triggers an App Runner deployment, waits for RUNNING, and
> finally smoke-tests the live `/health` endpoint — failing the build if it's not 200. CI is
> the build/push; CD is the deploy; the SHA tag gives traceability and rollback."

---

## Part 6 — Run the pipeline & view the Welcome Text

**6.1** Commit and push the workflow:
```powershell
git add .github/workflows/deploy.yml
git commit -m "Add AWS CI/CD pipeline"
git push
```

**6.2** GitHub → repo → **Actions** tab → click the running **"Build, Push & Deploy to AWS"**
run. Watch the 8 steps go green one by one.
✅ Expect: the final **Smoke test** step prints `✓ Live and healthy: https://…`.

**6.3** Open that App Runner URL in your browser.
✅ Expect: the **"👋 Welcome Text"** page — and the **Build:** line now shows the commit SHA,
proving this exact pipeline run deployed it.

**6.4 Prove the loop works:** edit the heading in `server.js` (e.g. `Welcome Text v2`),
commit, push. Within a few minutes the pipeline redeploys and the live page updates — that's
continuous deployment.

---

## Part 7 — Cleanup (do this to avoid charges)

```powershell
# Delete the App Runner service (stops billing)
aws apprunner delete-service --service-arn "<your-service-arn>" --region us-east-1

# Delete the ECR repo and its images
aws ecr delete-repository --repository-name welcome-aws --force --region us-east-1
```
Then in IAM, delete the `github-actions-welcome` access key (or the user) when finished.
✅ Expect: App Runner service status moves to `DELETED`; no further charges.

---

## What to say about this in an interview
> "I built a containerized web app and a GitHub Actions pipeline that tests it, builds a
> Docker image tagged with the commit SHA, pushes to Amazon ECR, triggers an App Runner
> deployment, waits for the service to reach RUNNING, then runs a smoke test against the
> live `/health` endpoint — failing the pipeline if it's not healthy. Every push to `main`
> redeploys automatically. Credentials are stored as GitHub secrets; in production I'd use
> GitHub OIDC with an IAM role instead of static keys."

Maps to resume lines: CI/CD with GitHub Actions ✓, AWS ✓, Docker ✓, secrets/env config ✓,
post-release validation ✓, automated deployment ✓.

---

## Appendix — Zero-cost alternative (S3 static website)

If you want to avoid all charges, deploy a static "Welcome Text" page to **S3** instead:

1. Create `index.html` with `<h1>Welcome Text</h1>`.
2. `aws s3 mb s3://welcome-<unique-name>` then enable static website hosting:
   `aws s3 website s3://welcome-<unique-name> --index-document index.html`
   (and add a public-read bucket policy).
3. Pipeline deploy step: `aws s3 sync . s3://welcome-<unique-name> --exclude ".git/*"`.
4. View at the S3 website endpoint URL.

Same pipeline shape (checkout → test → configure AWS → deploy → smoke test), no container
runtime cost. Trade-off: static only — no Docker/App Runner demonstration.

---

## Troubleshooting
| Symptom | Fix |
|---|---|
| `no basic auth credentials` on push | Re-run the `aws ecr get-login-password … \| docker login` command |
| App Runner stuck "Operation in progress" | Wait — first deploys take 3–5 min; check the service **Logs** tab |
| Page shows error / won't load | Port must be **8080** in both Dockerfile and App Runner config |
| Pipeline `AccessDenied` | IAM user missing ECR or App Runner permissions (Part 3.2) |
| `Unable to locate credentials` | The 3 AWS secrets are missing/misnamed in GitHub |
| Smoke test fails (non-200) | Open the URL manually; check App Runner logs; confirm `/health` returns OK |
| Unexpected AWS bill | Run Part 7 cleanup; App Runner bills while running |
