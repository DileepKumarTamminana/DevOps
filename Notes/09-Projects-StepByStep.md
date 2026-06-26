# 09 — Projects: Step-by-Step Execution Guide

Companion to [[07-HandsOn-Projects]]. That file explains *what* each project is; **this file
is the click-by-click "do exactly this" walkthrough**, written for Windows + PowerShell
(notes added where Linux/Mac differs).

> **Conventions in this guide**
> - `PS>` means type the command in **PowerShell**.
> - "✅ Expect:" tells you what success looks like.
> - "❌ If you see…" gives the fix for the most common error.
> - Replace anything in `<angle-brackets>` with your own value.

---

## Step 0 — One-time setup & verification

**0.1 Install the tools** (skip any you already have):
- Git → https://git-scm.com/download/win
- Node.js (LTS) → https://nodejs.org
- Docker Desktop → https://www.docker.com/products/docker-desktop/
- VS Code → https://code.visualstudio.com (you already have it)
- A GitHub account → https://github.com

**0.2 Verify each install** — open a **new** PowerShell window and run:
```powershell
git --version
node --version
npm --version
docker --version
```
✅ Expect: a version number from each (e.g. `git version 2.45.0`).
❌ If you see `not recognized`: the install didn't add it to PATH. Close and reopen
PowerShell; if still failing, reinstall and tick "Add to PATH".

**0.3 Start Docker Desktop** (needed for Projects 3, 4, 6). Launch it from the Start menu
and wait until the whale icon says "Engine running". Then:
```powershell
docker run hello-world
```
✅ Expect: "Hello from Docker!" message.

**0.4 Make a working folder** for all projects:
```powershell
cd "$HOME\Documents"
mkdir devops-projects
cd devops-projects
```

---

## Project 1 — Shell health script

**1.1** Create the folder and open it in VS Code:
```powershell
cd "$HOME\Documents\devops-projects"
mkdir p1-health; cd p1-health
code .
```

**1.2** In VS Code, create a new file `health.ps1` and paste:
```powershell
Write-Host "===== Health Report: $(Get-Date) ====="
Write-Host "`n--- Disk ---"
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free
Write-Host "`n--- Top 5 CPU processes ---"
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU
```

**1.3** Run it:
```powershell
./health.ps1
```
✅ Expect: a report with the date, your drives, and 5 processes.
❌ If you see `running scripts is disabled on this system`: allow local scripts once:
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
then run `./health.ps1` again and choose `Y`.

**1.4 (Bash version, optional)** Open the **Git Bash** terminal in VS Code (Terminal →
new terminal → pick "Git Bash" from the dropdown) and create `health.sh`:
```bash
#!/bin/bash
set -euo pipefail
echo "===== Health Report: $(date) ====="
echo "--- Disk ---"; df -h
echo "--- Top 5 CPU processes ---"; ps aux | head -6
```
Run: `bash health.sh`

✔️ **Checkpoint:** you can run a script and read its output.

---

## Project 2 — Git & branching + push to GitHub

**2.1** Set your Git identity once (use your GitHub email):
```powershell
git config --global user.name "Dileep Kumar Tamminana"
git config --global user.email "<your-github-email>"
```

**2.2** Create the project and first commit:
```powershell
cd "$HOME\Documents\devops-projects"
mkdir p2-git; cd p2-git
git init
"# My App" | Out-File -Encoding utf8 README.md
git add .
git commit -m "Initial commit"
```
✅ Expect: `1 file changed, 1 insertion(+)`.

**2.3** Practice the feature-branch flow:
```powershell
git checkout -b feature/add-greeting
"Hello DevOps" | Out-File -Encoding utf8 -Append README.md
git commit -am "Add greeting"
git checkout main
git merge feature/add-greeting
git branch -d feature/add-greeting
git log --oneline
```
✅ Expect: `git log` shows both commits on `main`.

**2.4** Create an empty repo on GitHub (UI): github.com → **New repository** → name it
`p2-git` → **do NOT** add a README → **Create**. Copy the repo URL.

**2.5** Connect and push:
```powershell
git remote add origin https://github.com/<your-username>/p2-git.git
git branch -M main
git push -u origin main
```
✅ Expect: refresh the GitHub page and see your files.
❌ If push asks for a password and rejects it: GitHub needs a **Personal Access Token**, not
your account password. Create one at github.com → Settings → Developer settings → Personal
access tokens → generate (scope: `repo`) and paste it as the password. (Or install
[GitHub CLI](https://cli.github.com) and run `gh auth login`.)

**2.6 (Stretch) Pull Request:** push a feature branch instead of merging locally:
```powershell
git checkout -b feature/readme-update
"More notes" | Out-File -Encoding utf8 -Append README.md
git commit -am "Update README"
git push -u origin feature/readme-update
```
Then on GitHub click **Compare & pull request** → **Create** → **Merge**.

✔️ **Checkpoint:** you can branch, merge, push, and open a PR.

---

## Project 3 — Dockerize a web app

**3.1** Create the project:
```powershell
cd "$HOME\Documents\devops-projects"
mkdir p3-docker; cd p3-docker
code .
```

**3.2** Create `server.js`:
```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  if (req.url === '/health') { res.writeHead(200); return res.end('OK'); }
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from a container!\n');
});
server.listen(3000, () => console.log('Listening on 3000'));
```

**3.3** Create `Dockerfile` (no extension):
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY server.js .
EXPOSE 3000
CMD ["node", "server.js"]
```

**3.4** Make sure Docker Desktop is running, then build:
```powershell
docker build -t hello-app:1.0 .
```
✅ Expect: ends with `naming to docker.io/library/hello-app:1.0` / `FINISHED`.
❌ If `Cannot connect to the Docker daemon`: Docker Desktop isn't started — launch it and wait.

**3.5** Run the container:
```powershell
docker run -d -p 8080:3000 --name hello hello-app:1.0
```
✅ Expect: a long container ID printed.
❌ If `port is already allocated`: change `-p 8081:3000` and use 8081 below.

**3.6** Test it:
```powershell
curl http://localhost:8080
curl http://localhost:8080/health
```
✅ Expect: `Hello from a container!` and `OK`. (Or just open http://localhost:8080 in a browser.)

**3.7** Inspect and clean up:
```powershell
docker ps                 # see it running
docker logs hello         # see "Listening on 3000"
docker stop hello
docker rm hello
```

✔️ **Checkpoint:** you built an image, ran it, and reached it from your browser.

---

## Project 4 — Docker Compose (app + database)

**4.1** Work inside the Project 3 folder (it reuses `server.js` + `Dockerfile`):
```powershell
cd "$HOME\Documents\devops-projects\p3-docker"
```

**4.2** Create `docker-compose.yml`:
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

**4.3** Start both services:
```powershell
docker compose up -d
```
✅ Expect: `Started` for both `web` and `db`.

**4.4** Verify:
```powershell
docker compose ps          # both should be "running"/"healthy"
curl http://localhost:8080
```

**4.5** Tear down (use `-v` to also delete the DB volume):
```powershell
docker compose down
```

✔️ **Checkpoint:** two containers started together with one command.

---

## Project 5 — First CI pipeline (GitHub Actions)

**5.1** Create a repo with the app and push it (reuse the Project 3 app):
```powershell
cd "$HOME\Documents\devops-projects"
mkdir p5-ci; cd p5-ci
Copy-Item "..\p3-docker\server.js" .
Copy-Item "..\p3-docker\Dockerfile" .
git init
git add .
git commit -m "Add app"
```
Create an empty `p5-ci` repo on GitHub, then:
```powershell
git remote add origin https://github.com/<your-username>/p5-ci.git
git branch -M main
git push -u origin main
```

**5.2** Create the workflow file. The folder path matters exactly:
```powershell
mkdir -p .github/workflows
```
Create `.github/workflows/ci.yml`:
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
      - run: node -e "console.log('build OK')"
      - name: Check app file exists
        run: test -f server.js && echo "server.js exists" || exit 1
```

**5.3** Commit and push to trigger it:
```powershell
git add .github
git commit -m "Add CI workflow"
git push
```

**5.4** Watch it run: GitHub → your repo → **Actions** tab → click the running workflow.
✅ Expect: a green check ✓ next to "CI" after ~30s.
❌ If it's red: click the failed step to read the log — usually a YAML indentation error.
Paste your YAML into https://www.yamllint.com to check.

✔️ **Checkpoint:** code pushed → pipeline ran automatically.

---

## Project 6 — CI/CD: build & push a Docker image

**6.1** Create a Docker Hub account → https://hub.docker.com, then create an **Access
Token**: Account Settings → Security → **New Access Token** → copy it (you'll see it once).

**6.2** In your `p5-ci` GitHub repo → **Settings → Secrets and variables → Actions →
New repository secret**. Add two:
- `DOCKERHUB_USER` = your Docker Hub username
- `DOCKERHUB_TOKEN` = the token you copied

**6.3** Add `.github/workflows/docker.yml`:
```yaml
name: Build & Push Image
on:
  push: { branches: [ main ] }
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKERHUB_USER }}/hello-app:latest
```

**6.4** Commit and push:
```powershell
git add .github/workflows/docker.yml
git commit -m "Add image build & push workflow"
git push
```

**6.5** Watch **Actions** → "Build & Push Image".
✅ Expect: green ✓, and the image appears at `hub.docker.com/r/<user>/hello-app`.
❌ If login fails: re-check the secret names match exactly and the token wasn't truncated.

✔️ **Checkpoint:** a push automatically built and published a container image.

---

## Project 7 — Deploy to Azure + post-release smoke test

> Needs a free Azure account (https://azure.microsoft.com/free) and the Azure CLI
> (https://aka.ms/installazurecliwindows). Verify: `az --version`.

**7.1** Log in and create the infrastructure:
```powershell
az login
az group create -n rg-hello -l eastus
az appservice plan create -g rg-hello -n plan-hello --is-linux --sku B1
az webapp create -g rg-hello -p plan-hello -n hello-<unique-name> `
  --deployment-container-image-name <dockerhubuser>/hello-app:latest
```
> In PowerShell the backtick `` ` `` continues a command onto the next line. `<unique-name>`
> must be globally unique (e.g. `hello-dileep-2026`).

**7.2** Tell the web app which port the container uses:
```powershell
az webapp config appsettings set -g rg-hello -n hello-<unique-name> `
  --settings WEBSITES_PORT=3000
```

**7.3** Browse to `https://hello-<unique-name>.azurewebsites.net` (first load can take a
minute while it pulls the image).
✅ Expect: "Hello from a container!"

**7.4** Add an automated smoke test to your pipeline — append to `docker.yml` after the
build step:
```yaml
      - name: Smoke test
        run: |
          url="https://hello-<unique-name>.azurewebsites.net/health"
          status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
          if [ "$status" -eq 200 ]; then echo "Healthy ✓"; else echo "FAILED ($status)"; exit 1; fi
```
Commit & push, then watch the run fail or pass based on the live health check.

**7.5 Clean up to avoid charges** when done:
```powershell
az group delete -n rg-hello --yes --no-wait
```
✅ Expect: deleting the resource group removes everything you created.

✔️ **Checkpoint:** a full path — code → CI → image → cloud → automated validation.

---

## Project 8 — Terraform IaC (advanced, optional)

**8.1** Install Terraform (https://developer.hashicorp.com/terraform/install) → verify
`terraform --version`. You'll reuse the Azure login from Project 7 (`az login`).

**8.2** Create `main.tf`:
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

**8.3** Run the IaC lifecycle:
```powershell
terraform init      # downloads the provider
terraform plan      # preview: shows "1 to add"
terraform apply     # type 'yes' to create
terraform destroy   # type 'yes' to remove
```
✅ Expect: `plan` shows what will change *before* anything happens — this is the power of IaC.

✔️ **Checkpoint:** you created and destroyed cloud infra from a file, repeatably.

---

## Capstone — assemble the portfolio repo

1. Make one new repo `devops-capstone`.
2. Put in it: `server.js`, `Dockerfile`, `docker-compose.yml`, and `.github/workflows/`
   with **both** the CI workflow and the build-push-deploy-smoketest workflow.
3. Add a `README.md` that explains the architecture and links the live URL.
4. Add a one-page `ROLLBACK.md` describing how you'd revert a bad deploy.
5. Pin this repo on your GitHub profile and link it on your resume.

> This single repo demonstrates CI, CD, Docker, cloud deploy, secrets management, and
> post-release validation — your whole skill set, provable in one link.

---

## General troubleshooting cheat-sheet
| Symptom | Likely cause / fix |
|---|---|
| `not recognized as a command` | Tool not in PATH — reopen PowerShell / reinstall |
| `running scripts is disabled` | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| `Cannot connect to the Docker daemon` | Start Docker Desktop, wait for "Engine running" |
| `port is already allocated` | Change the host port (`-p 8081:3000`) |
| GitHub push asks for password & fails | Use a Personal Access Token, not your password |
| Actions workflow not appearing | File must be in `.github/workflows/` and be valid YAML |
| YAML errors | Check indentation (spaces, not tabs) at yamllint.com |
| Azure web app shows "Application Error" | Set `WEBSITES_PORT=3000`; check `az webapp log tail` |
| Unexpected cloud charges | `az group delete -n <rg> --yes` to remove everything |
