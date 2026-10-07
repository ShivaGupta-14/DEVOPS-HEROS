# GitOps

## What is GitOps?

GitOps is a way of deploying and managing infra/apps where **Git holds the desired state** of the system, and an agent running in the cluster keeps the real system the same as Git. To change anything we change Git (commit / PR), not the cluster.

Tools: Argo CD, Flux.

## Git as the source of truth

- The YAML in the repo is what *should* be running. If the cluster and Git are different, Git wins.
- Every change is a commit, so we get history, review by PR, and who changed what for free.
- Rollback = `git revert`.
- If a cluster is lost, we can point a new cluster to the same repo and get everything back.
- Nobody needs `kubectl apply` access to prod, only the GitOps agent does.

## Declarative configuration

We describe *what* we want, not the steps to get there.

```yaml
spec:
  replicas: 3          # "I want 3 pods"
```

vs imperative: `kubectl scale --replicas=3`, `kubectl run ...`. Imperative commands are not saved anywhere and can't be reviewed. Kubernetes, Helm and Terraform are all declarative, which is why GitOps fits them.

## Continuous reconciliation

The agent runs a loop all the time:

```text
desired state (Git)  --compare--  actual state (cluster)
                         |
                  different? -> sync (apply Git to cluster)
```

- **Drift**: someone changes the cluster by hand (e.g. `kubectl scale`). Argo CD marks it OutOfSync.
- **selfHeal: true**: Argo CD puts it back to what Git says automatically.
- **prune: true**: if a file is deleted from Git, the resource is deleted from the cluster.

Argo CD checks the repo every ~3 minutes by default (or instantly with a webhook).

## GitOps workflow

```text
Developer -> change YAML -> commit/PR -> review + merge -> Git (main)
                                                            |
                                         Argo CD pulls & detects change
                                                            |
                                         sync to Kubernetes -> app updated
```

With CI: CI builds and pushes the image, then updates the image tag in the GitOps repo. CD is done by Argo CD pulling, not by CI pushing into the cluster (pull model, cluster credentials stay inside the cluster).

## Kubernetes + GitOps

- Argo CD is installed in the cluster (`argocd` namespace).
- An `Application` object tells it: which repo, which path, which branch, and which cluster/namespace to deploy to.
- It can deploy plain YAML, Kustomize or Helm charts.
- It shows each app's **Sync status** (Synced / OutOfSync) and **Health** (Healthy / Progressing / Degraded).

Demo for this is in [Homework.md](../Homework.md) using the repo https://github.com/ShivaGupta-14/session20-gitops.
