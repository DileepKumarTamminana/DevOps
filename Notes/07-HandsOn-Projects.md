# 07 — Hands-On Projects (Simple, Runnable)

Learning sticks when you build. These projects go from absolute beginner to a small
end-to-end DevOps pipeline. Each lists **goal · what you'll learn · steps · stretch goals**.
Do them in order — later ones reuse earlier ones.

> **Setup once:** Install [Git](https://git-scm.com), [Docker Desktop](https://www.docker.com/products/docker-desktop/),
> [Node.js](https://nodejs.org) (or Python), and a free [GitHub](https://github.com) account.
> A free [Azure](https://azure.microsoft.com/free/) and/or [AWS](https://aws.amazon.com/free/) account for the cloud projects.

---

## Project 1 — Your first shell script (Bash + PowerShell)
**Goal:** automate a small repetitive task. **Learn:** [[05-Scripting-and-Query]].

A "system health" script that prints disk, memory, and the top processes.

**Bash** — `health.sh`:
```bash
#!/bin/bash
set -euo pipefail
echo "===== Health Report: $(date) ====="
echo "--- Disk ---";   df -h | grep -vE '^tmpfs'
echo "--- Memory ---"; free -h
echo "--- Top 5 CPU processes ---"
ps aux --sort=-%cpu | head -6
```
Run: `chmod +x health.sh && ./health.sh`

**PowerShell** — `health.ps1`:
```powershell
Write-Host "===== Health Report: $(Get-Date) ====="
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU
```
Run: `./health.ps1`

**Stretch:** append output to a log file with a timestamp; schedule it (cron / Task Scheduler).

---

## Project 2 — Git & branching workflow
**Goal:** practice the everyday Git flow. **Learn:** branching strategy ([[03-Infrastructure-and-Config]]).

```bash
git init my-app && cd my-app
echo "# My App" > README.md
git add . && git commit -m "Initial commit"

# Feature branch workflow (GitHub Flow)
git checkout -b feature/add-greeting
echo "Hello DevOps" >> README.md
git commit -am "Add greeting"
git checkout main
git merge feature/add-greeting
git branch -d feature/add-greeting
```
**Stretch:** push to GitHub, open a Pull Request from a feature branch, merge it in the UI.
Create a merge conflict on purpose and resolve it.

---

## Project 3 — Dockerize a simple web app
**Goal:** package an app in a container. **Learn:** Docker ([[03-Infrastructure-and-Config]]).

`server.js` (tiny Node app):
```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.url === '/health') { res.writeHead(200); return res.end('OK'); }
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from a container!\n');
});
server.listen(3000, () => console.log('Listening on 3000'));
```
`Dockerfile`:
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```
Build & run:
```bash
docker build -t hello-app:1.0 .
docker run -d -p 8080:3000 --name hello hello-app:1.0
curl http://localhost:8080          # -> Hello from a container!
curl http://localhost:8080/health   # -> OK
docker logs hello
docker stop hello && docker rm hello
```
**Stretch:** convert to a multi-stage build; push the image to Docker Hub.

---

## Project 4 — Multi-container app with Docker Compose
**Goal:** run an app + database together. **Learn:** service orchestration, env config.

`docker-compose.yml`:
```yaml
services:
  web:
    build: .
    ports:
      - "8080:3000"
    environment:
      - DB_HOST=db
    depends_on:
      - db
  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_PASSWORD=devpass
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```
Run: `docker compose up -d` · Stop: `docker compose down`
**Stretch:** add a Redis cache service; connect the web app to the DB.

---

## Project 5 — Your first CI pipeline (GitHub Actions)
**Goal:** auto-build and test on every push. **Learn:** [[01-CICD-and-Automation]].

In your repo, create `.github/workflows/ci.yml`:
```yaml
name: CI
on:
  push: { branches: [ main ] }
  pull_request: { branches: [ main ] }
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci || echo "no deps yet"
      - run: node -e "console.log('build OK')"
      - name: Run a test
        run: |
          test -f server.js && echo "server.js exists ✓" || (echo "missing!" && exit 1)
```
Commit, push, and watch it run under the repo's **Actions** tab.
**Stretch:** add a matrix (`node: [18, 20, 22]`); add a status badge to the README.

---

## Project 6 — CI/CD: build a Docker image and push it
**Goal:** automate image builds. **Learn:** secrets, registries, CD.

`.github/workflows/docker.yml`:
```yaml
name: Build & Push Image
on:
  push: { branches: [ main ] }
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USER }}/hello-app:latest
```
Add `DOCKERHUB_USER` and `DOCKERHUB_TOKEN` under **Settings → Secrets → Actions**.
**Stretch:** tag images with the git SHA; add a manual `workflow_dispatch` trigger.

---

## Project 7 — Deploy to the cloud + post-release health check
**Goal:** a real end-to-end deploy with validation. **Learn:** [[02-Cloud-Platforms]], [[06-Practices]].

Deploy the container to **Azure App Service** (or AWS App Runner), then add a smoke test.

Azure CLI quick path:
```bash
az login
az group create -n rg-hello -l eastus
az appservice plan create -g rg-hello -n plan-hello --is-linux --sku B1
az webapp create -g rg-hello -p plan-hello -n hello-<unique> \
  --deployment-container-image-name <dockerhubuser>/hello-app:latest
```
Add a post-deploy smoke test step to your workflow:
```yaml
      - name: Smoke test
        run: |
          status=$(curl -s -o /dev/null -w "%{http_code}" https://hello-<unique>.azurewebsites.net/health)
          [ "$status" -eq 200 ] && echo "Healthy ✓" || (echo "FAILED ($status)"; exit 1)
```
**Stretch:** add a `production` environment with a required approval before deploy;
write a one-page rollback plan.

---

## Project 8 — Infrastructure as Code (stretch / advanced)
**Goal:** create cloud infra from a file, not clicks. **Learn:** IaC.

Minimal **Terraform** to create an Azure resource group:
```hcl
terraform {
  required_providers { azurerm = { source = "hashicorp/azurerm", version = "~>3.0" } }
}
provider "azurerm" { features {} }

resource "azurerm_resource_group" "rg" {
  name     = "rg-terraform-demo"
  location = "East US"
}
```
Run: `terraform init` → `terraform plan` → `terraform apply` → `terraform destroy`.
**Stretch:** add a storage account; store state remotely; parameterise with variables.

---

## Suggested portfolio capstone
Combine projects 3 → 7 into one repo:
> A containerized web app, built and tested by GitHub Actions, image pushed to a registry,
> deployed to the cloud, with an automated post-deploy health check and a documented
> rollback plan.

That single repo demonstrates **CI, CD, Docker, cloud, secrets, and validation** — i.e.
your whole resume, provable in a link.

---

## Project progress tracker
- [ ] P1 — Shell health script (Bash + PowerShell)
- [ ] P2 — Git branching workflow + PR
- [ ] P3 — Dockerized web app
- [ ] P4 — Docker Compose (app + DB)
- [ ] P5 — CI pipeline (build + test)
- [ ] P6 — CI/CD image build & push
- [ ] P7 — Cloud deploy + smoke test
- [ ] P8 — Terraform IaC
- [ ] Capstone repo assembled & linked on resume
