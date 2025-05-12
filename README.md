# FluxCD Guide with EKS Setup and Flagger Integration

---

## Segment 1: Theory

### What is FluxCD?

FluxCD is a GitOps-based deployment tool for Kubernetes that automates the synchronization of manifests from a Git repository into your cluster. It supports continuous delivery and focuses on being a pull-based system, where the cluster periodically checks Git for updates and reconciles the desired state.

### Key Concepts:

- **GitOps**: A methodology where Git is the single source of truth for system configuration.
- **Reconciliation Loop**: FluxCD runs in a loop, ensuring the cluster matches the config in Git.
- **Bootstrap**: Initializes a Git repository to be tracked and managed by FluxCD.
- **Controllers**: Flux consists of controllers like `source-controller`, `kustomize-controller`, etc., for modular resource handling.
- **Flagger**: A progressive delivery tool that integrates with Flux and service meshes to automate canary deployments.

---

### FluxCD vs ArgoCD

| Feature               | FluxCD                                      | ArgoCD                                     |
|-----------------------|---------------------------------------------|--------------------------------------------|
| UI                    | No built-in UI (CLI/GitHub only)            | Rich Web UI for visualization              |
| Installation Size     | Lightweight                                 | Slightly heavier due to the UI             |
| GitOps Approach       | Pull-based                                  | Pull-based (with optional push via webhook)|
| Kubernetes Native     | Highly native with GitOps Toolkit           | Also very Kubernetes-native                |
| Multi-Repo Support    | Good                                         | Excellent with UI filtering                |
| Canary Deployments    | Works well with Flagger                     | Supported via Argo Rollouts                |
| Secrets Management    | Integrates with SOPS, Mozilla Sealed Secrets| Integrates with Vault, Bitnami Sealed Secrets |
| Rollback              | Manual or via Flagger                       | UI-supported rollback                      |

#### ✅ Advantages of FluxCD:
- Lightweight, modular, and ideal for CLI/git-based workflows.
- Git-first approach with better Git integration.
- Excellent for automation and scripting in CI/CD pipelines.
- Easier to extend via its GitOps Toolkit.

#### ❌ Disadvantages of FluxCD:
- Lacks built-in GUI for visibility.
- Learning curve for newcomers due to modular controller setup.
- Debugging can be CLI-intensive.

---

## Segment 2: Practical: EKS Cluster with FluxCD and Flagger Setup

### a) Create an EKS Cluster using `eksctl`

```bash
eksctl create cluster --name flux \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 2 \
  --managed \
  --kubeconfig ~/.kube/config
```

**Explanation:**

- Creates a new EKS cluster named `flux` in the `us-east-1` region.
- Uses `t3.medium` nodes with one node initially.
- Cluster scales between 1 and 2 nodes automatically.
- `--managed` provisions nodes via AWS-managed node groups.
- Updates `~/.kube/config` with the new cluster context.

---

### b) Pre-Check the Cluster for Flux Compatibility

```bash
flux check --pre
```

**Explanation:**

- Validates whether your Kubernetes cluster meets the minimum requirements for FluxCD installation.
- Checks for permissions, controller readiness, etc.

---

### c) Install FluxCD into the Cluster

```bash
flux install
```

**Explanation:**

- Installs the necessary Flux controllers (`source-controller`, `kustomize-controller`, etc.) into the `flux-system` namespace.
- Sets up the GitOps environment inside your cluster.

---

### d) Verify the FluxCD Pods

```bash
kubectl get pods -n flux-system
```

**Explanation:**

- Ensures that Flux controllers are running correctly inside the `flux-system` namespace.

---

### e) Bootstrap Git Repository with Flux

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$REPO_NAME \
  --branch=main \
  --path=clusters/my-cluster \
  --personal
```

**Explanation:**

- Connects your cluster to a GitHub repo and sets up initial configurations.
- Flux starts watching the specified Git repo (`$REPO_NAME`) under `clusters/my-cluster` directory on the `main` branch.
- `--personal` means it's a user-owned repo (not under a GitHub organization).

---

### f) Install Flagger with Prometheus (for progressive delivery)

```bash
helm upgrade -i flagger flagger/flagger \
  --namespace flagger-system \
  --create-namespace \
  --set prometheus.install=true \
  --set meshProvider=kubernetes
```

**Explanation:**

- Installs Flagger using Helm into the `flagger-system` namespace.
- `--create-namespace` creates the namespace if it doesn't exist.
- `prometheus.install=true` installs Prometheus for metric collection (useful for testing).
- `meshProvider=kubernetes` configures Flagger for native Kubernetes deployments (no service mesh like Istio needed).

---

### Demo Repository (Optional)

You can use the following GitHub repo to test your Flux setup:

```
https://github.com/ar-ram/fluxCD.git
```

> This repo can be forked and bootstrapped using the above `flux bootstrap` command to test real GitOps workflows.

---