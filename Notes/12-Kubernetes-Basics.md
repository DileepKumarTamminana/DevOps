# 12 — Kubernetes & Container Orchestration (Basics)

Docker ([[03-Infrastructure-and-Config]]) runs **one** container on **one** machine.
Real systems run **many** containers across **many** machines, and need them to self-heal,
scale, and update without downtime. That's what an **orchestrator** does — and Kubernetes
(K8s) is the industry standard.

> Even if your current role uses App Service / App Runner instead of K8s, interviewers
> often ask the basics. Aim to *explain the concepts*, not memorise every flag.

---

## 1. Why orchestration exists (the "why")

Running containers manually doesn't scale. You'd have to, by hand:
- restart containers that crash,
- spread them across servers,
- replace them on a new release with no downtime,
- route traffic to healthy ones only,
- scale up under load and down when quiet.

**Kubernetes automates all of this.** You declare the *desired state* ("run 3 copies of this
image"), and K8s continuously works to make reality match — that's the **reconciliation loop**.

> **Interview-ready:** "Kubernetes is a container orchestrator. I declare desired state and
> it self-heals, scales, load-balances, and rolls out updates to match — across a cluster of
> machines."

---

## 2. Cluster architecture

```
            ┌──────────────── Control Plane (the "brain") ────────────────┐
            │  API Server · Scheduler · Controller Manager · etcd (store)  │
            └───────────────────────────┬──────────────────────────────────┘
                                         │ instructs
        ┌────────────────────────────────┼────────────────────────────────┐
        ▼                                 ▼                                 ▼
   ┌─────────┐                       ┌─────────┐                      ┌─────────┐
   │  Node   │                       │  Node   │                      │  Node   │   ← worker machines
   │ ┌─────┐ │                       │ ┌─────┐ │                      │ ┌─────┐ │
   │ │ Pod │ │                       │ │ Pod │ │                      │ │ Pod │ │   ← Pods run containers
   │ └─────┘ │                       │ └─────┘ │                      │ └─────┘ │
   │ kubelet │                       │ kubelet │                      │ kubelet │   ← agent on each node
   └─────────┘                       └─────────┘                      └─────────┘
```

| Component | Role |
|---|---|
| **Control plane** | Manages the cluster, makes scheduling decisions |
| **API Server** | The front door — all commands go through it (`kubectl` talks to this) |
| **etcd** | Key-value store holding the cluster's entire state |
| **Scheduler** | Decides which node a new Pod runs on |
| **Controller Manager** | Runs the reconciliation loops (keeps desired = actual) |
| **Node** | A worker machine (VM/physical) that runs Pods |
| **kubelet** | Agent on each node that starts/stops containers |
| **kube-proxy** | Handles networking/routing on each node |

---

## 3. Core objects (the building blocks)

| Object | What it is |
|---|---|
| **Pod** | Smallest unit — one (or a few tightly-coupled) containers sharing network/storage |
| **ReplicaSet** | Keeps N identical Pods running |
| **Deployment** | Manages ReplicaSets; handles rolling updates & rollbacks (what you usually create) |
| **Service** | Stable network endpoint + load balancing across a set of Pods |
| **Ingress** | HTTP/HTTPS routing into the cluster (URLs → Services) |
| **ConfigMap** | Non-secret configuration injected into Pods |
| **Secret** | Sensitive values (base64-encoded; integrate with a real vault for production) |
| **Namespace** | Logical partition of a cluster (e.g. `dev`, `prod`) |
| **Volume / PVC** | Persistent storage for stateful Pods |

> **Pod vs Container:** a Pod *wraps* one or more containers. You scale and schedule Pods,
> not individual containers.
> **Deployment vs Pod:** you almost never create bare Pods — you create a Deployment, which
> creates and manages the Pods for you (and replaces them if they die).

---

## 4. A minimal Deployment + Service (YAML)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  replicas: 3                 # desired state: 3 Pods
  selector:
    matchLabels: { app: hello }
  template:
    metadata:
      labels: { app: hello }
    spec:
      containers:
        - name: hello
          image: <user>/hello-app:latest
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector: { app: hello }    # routes to Pods with this label
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer          # exposes an external IP
```

The **Service `selector`** matches the Pod **labels** — that's how a Service knows which
Pods to send traffic to. This label/selector pattern is everywhere in K8s.

---

## 5. Essential `kubectl` commands

`kubectl` is the CLI that talks to the cluster's API server.
```bash
kubectl apply -f deploy.yaml      # create/update from a manifest (declarative)
kubectl get pods                   # list Pods
kubectl get pods -o wide           # + which node they're on
kubectl get deployments,svc        # list multiple object types
kubectl describe pod <name>        # detailed status + events (great for debugging)
kubectl logs <pod>                 # view a Pod's logs
kubectl logs -f <pod>              # follow logs live
kubectl exec -it <pod> -- sh       # shell into a Pod
kubectl scale deployment hello --replicas=5    # scale up/down
kubectl rollout status deployment hello        # watch a rollout
kubectl rollout undo deployment hello          # roll back to previous version
kubectl delete -f deploy.yaml      # remove what the manifest created
kubectl get events --sort-by=.lastTimestamp   # recent cluster events
```

> **Debugging a broken Pod (interview favourite):** `kubectl get pods` → see status
> (`CrashLoopBackOff`, `ImagePullBackOff`, `Pending`) → `kubectl describe pod` for events →
> `kubectl logs` for app errors.

| Pod status | Usual meaning |
|---|---|
| `Pending` | Can't be scheduled (no resources / node) |
| `ImagePullBackOff` | Can't pull the image (wrong name/tag or no registry auth) |
| `CrashLoopBackOff` | Container keeps crashing on start (check logs) |
| `Running` | Healthy |

---

## 6. How K8s gives you DevOps superpowers

- **Self-healing** — crashed Pods are restarted; dead nodes' Pods are rescheduled.
- **Rolling updates & rollback** — Deployments replace Pods gradually (zero-downtime) and can
  instantly roll back (`kubectl rollout undo`). Ties to release strategies in
  [[03-Infrastructure-and-Config]].
- **Horizontal scaling** — manual (`kubectl scale`) or automatic (**HPA**, Horizontal Pod
  Autoscaler, based on CPU/metrics).
- **Health probes** — `livenessProbe` (restart if unhealthy) and `readinessProbe` (don't send
  traffic until ready) — the K8s version of post-release validation ([[06-Practices]]).
- **Service discovery & load balancing** — Services give stable names + spread traffic.

### Managed Kubernetes (you rarely run the control plane yourself)
| Cloud | Service |
|---|---|
| Azure | **AKS** (Azure Kubernetes Service) |
| AWS | **EKS** (Elastic Kubernetes Service) |
| Google | **GKE** |

Managed services run the control plane for you; you just manage workloads.

### Helm (package manager for K8s)
**Helm** packages a set of K8s manifests as a reusable, versioned **chart** with
configurable values — like `apt`/`npm` but for Kubernetes apps. Mention it as "how teams
template and reuse K8s deployments."

> **Interview-ready:** "A Deployment keeps N Pods of my app running and does zero-downtime
> rolling updates I can roll back. A Service load-balances stable traffic to them. Liveness
> and readiness probes keep only healthy Pods in rotation. In the cloud I'd use AKS/EKS so I
> don't manage the control plane, and Helm to template deployments."

---

## 7. Try it locally (optional hands-on)
1. Enable Kubernetes in **Docker Desktop** (Settings → Kubernetes → Enable), or install
   [minikube](https://minikube.sigs.k8s.io) / [kind](https://kind.sigs.k8s.io).
2. Use the `hello-app` image from [[10-AWS-Pipeline-Project]] / Project 3.
3. `kubectl apply -f deploy.yaml` → `kubectl get pods` → `kubectl scale … --replicas=5`.
4. Delete a Pod (`kubectl delete pod <name>`) and watch K8s recreate it (self-healing).

---

## Practice checklist
- [ ] Explain Pod vs Deployment vs Service in one sentence each.
- [ ] Draw the control-plane / node architecture from memory.
- [ ] Write a minimal Deployment YAML with 3 replicas.
- [ ] Debug a `CrashLoopBackOff` using `describe` + `logs`.
- [ ] Do a rolling update and a `rollout undo`.
- [ ] Name the AKS/EKS/GKE managed services and what Helm is for.
