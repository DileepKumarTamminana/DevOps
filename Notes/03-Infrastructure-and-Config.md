# 03 — Infrastructure & Configuration

Covers: Docker · Linux administration · Environment configuration · Release management

---

## 1. Docker & containers

### The problem containers solve
*"It works on my machine"* — software behaves differently across environments because
of OS, library, and config differences. **Containers** package an app with everything
it needs to run, so it behaves identically everywhere.

### Container vs Virtual Machine
| | Virtual Machine | Container |
|---|---|---|
| Includes | Full guest OS | Just app + dependencies |
| Size | GBs | MBs |
| Startup | Minutes | Seconds |
| Isolation | Strong (hardware-level) | Process-level (shares host kernel) |

Containers are lighter because they **share the host OS kernel** instead of bundling a full OS.

### Key Docker concepts
| Term | Meaning |
|---|---|
| **Image** | A read-only template (the "recipe" + ingredients) |
| **Container** | A running instance of an image |
| **Dockerfile** | Text file with instructions to build an image |
| **Registry** | Store/share images (Docker Hub, Azure Container Registry, ECR) |
| **Volume** | Persistent storage outside the container's lifecycle |

### A basic Dockerfile
```dockerfile
# Base image
FROM node:20-alpine

# Set working directory inside the container
WORKDIR /app

# Copy dependency manifests first (layer caching!)
COPY package*.json ./
RUN npm ci

# Copy the rest of the app
COPY . .

# Document the port the app listens on
EXPOSE 3000

# Command run when the container starts
CMD ["node", "server.js"]
```

### Essential Docker commands
```bash
docker build -t myapp:1.0 .        # build image from Dockerfile
docker images                       # list images
docker run -d -p 8080:3000 myapp:1.0   # run, map host:container ports, detached
docker ps                           # list running containers
docker ps -a                        # include stopped containers
docker logs <container_id>          # view logs
docker exec -it <container_id> sh   # shell into a running container
docker stop <container_id>          # stop
docker rm <container_id>            # remove container
docker rmi <image_id>               # remove image
docker push myregistry/myapp:1.0    # push to registry
```

### Concepts to be aware of
- **Layer caching** — each Dockerfile instruction is a cached layer; order matters
  (copy dependency files before source so code changes don't bust the dependency cache).
- **Multi-stage builds** — build in one stage, copy only the result into a small final
  image → smaller, more secure images.
- **docker-compose** — define multi-container apps (app + db + cache) in one `docker-compose.yml`.
- **Don't bake secrets into images** — pass them at runtime via env vars / secret stores.

> **Interview-ready:** "An image is the template, a container is a running instance.
> Containers share the host kernel, making them lighter than VMs. I order Dockerfile steps
> to maximise layer caching and use multi-stage builds to keep images small."

---

## 2. Linux administration

Most servers and CI runners are Linux, so command-line fluency is essential.

### Filesystem & navigation
```bash
pwd                 # print working directory
ls -la              # list all (incl. hidden) with details
cd /var/log         # change directory
find / -name "*.log"        # find files
tree                # directory tree
```

### File operations & viewing
```bash
cat file.txt        # print whole file
less file.txt       # scrollable view
head -n 20 file     # first 20 lines
tail -n 50 file     # last 50 lines
tail -f app.log     # follow live (great for logs!)
grep "ERROR" app.log        # search text
grep -ri "timeout" /etc     # recursive, case-insensitive
cp src dest         # copy
mv old new          # move/rename
rm -rf dir          # delete (careful!)
```

### Permissions (very common interview topic)
```bash
ls -l               # shows  -rwxr-xr--
```
- Format: `[type][owner][group][others]`, e.g. `rwx r-x r--`.
- `r`=4, `w`=2, `x`=1. So `chmod 755 file` = owner rwx(7), group rx(5), others rx(5).
```bash
chmod 644 file              # rw-r--r--
chmod +x script.sh          # make executable
chown user:group file       # change ownership
```

### Processes & system
```bash
ps aux | grep nginx         # find a process
top  /  htop                # live resource usage
kill -9 <pid>               # force kill a process
df -h                       # disk space
du -sh /var/log             # size of a directory
free -h                     # memory usage
systemctl status nginx      # service status (systemd)
systemctl restart nginx     # restart a service
journalctl -u nginx -f      # follow a service's logs
```

### Networking
```bash
curl -I https://site.com    # HTTP headers
ping host                   # connectivity
netstat -tulpn  /  ss -tulpn   # listening ports
nslookup / dig domain.com   # DNS lookup
```

### Package management
- **Debian/Ubuntu:** `apt update && apt install <pkg>`
- **RHEL/CentOS:** `yum install <pkg>` or `dnf install <pkg>`

> **Interview-ready:** "I use `tail -f` and `journalctl` for live logs, `chmod`/`chown`
> for permissions (rwx = 4/2/1), `systemctl` for services, and `df -h`/`top` for disk and
> resource checks during incidents."

---

## 3. Environment configuration

DevOps separates **code** (same everywhere) from **configuration** (differs per environment).

### Typical environments
| Env | Purpose |
|---|---|
| **Dev** | Developers integrate and test new code |
| **UAT / Staging** | Business/QA validation; mirrors production |
| **Production (Prod)** | Live system used by real users |

### How config is managed
- **Environment variables** — inject values at runtime (`API_URL`, `DB_HOST`).
- **Config files** — `appsettings.json`, `.env`, YAML — often one per environment.
- **Secrets management** — never store secrets in code/config files. Use:
  - Azure Key Vault, AWS Secrets Manager
  - GitHub Actions secrets / Azure DevOps variable groups
- **The 12-Factor principle:** *store config in the environment*, not in code, so the
  same build artifact can be promoted across Dev → UAT → Prod by only changing config.

> **Interview-ready:** "I build an artifact once and promote it across environments,
> changing only environment-specific config and secrets, which live in a vault — never
> in source control."

---

## 4. Release management

The discipline of **planning, scheduling, and controlling** software releases so they
reach production safely.

### Key activities
- **Versioning** — e.g. **Semantic Versioning** `MAJOR.MINOR.PATCH` (2.4.1).
- **Branching strategy** — decide how code flows to release (see below).
- **Release/deployment approvals** — gates requiring sign-off before Prod.
- **Pre-deployment checklist** — backups taken, rollback plan ready, stakeholders notified.
- **Rollback plan** — how to revert quickly if the release fails.
- **Post-release validation** — smoke tests after deploy (see [[06-Practices]]).

### Branching strategies
| Strategy | How it works | Best for |
|---|---|---|
| **GitHub Flow** | `main` + short-lived feature branches; deploy from main | Continuous delivery |
| **Git Flow** | `main`, `develop`, `feature/*`, `release/*`, `hotfix/*` | Scheduled releases |
| **Trunk-based** | Everyone commits to trunk frequently, behind feature flags | High-velocity teams |

### Deployment strategies (reduce risk / downtime)
| Strategy | How | Benefit |
|---|---|---|
| **Blue-Green** | Two identical environments; switch traffic to the new one | Instant rollback |
| **Canary** | Release to a small % of users first | Limit blast radius |
| **Rolling** | Update servers in batches | No full downtime |
| **Recreate** | Stop old, start new | Simple but causes downtime |

### Cutover (from your resume)
A **cutover** is the planned switch from an old system/version to a new one, often during
a migration. A good cutover has: a detailed runbook, a timeline, defined roles, a
**rollback plan**, and a go/no-go decision point. Goal: **zero (or minimal) downtime**.

> **Interview-ready:** "Release management is about getting changes to production safely:
> a branching strategy, approval gates, a pre-deployment checklist, and always a rollback
> plan. Blue-green and canary deployments minimise risk and downtime."

---

## Practice checklist
- [ ] Write a Dockerfile, build it, and run the container mapping a port.
- [ ] Use `tail -f` to follow a log and `grep` to filter errors.
- [ ] Explain `chmod 755` and `chmod 644` from memory.
- [ ] Externalise a config value into an environment variable.
- [ ] Diagram a blue-green deployment and where rollback happens.
