# Session 10 - Pods, ReplicaSets & Deployments Homework

Everything is done in namespace `s10` on minikube. Since NodePort is not reachable directly on Mac (docker driver), I used a `curl` pod inside the cluster to hit the services.

```bash
kubectl create ns s10
kubectl config set-context --current --namespace=s10
kubectl run curl --image=curlimages/curl -- sleep infinity
```

# Task 1: Deployment Strategies

## 01. Rolling Update

YAML: [deployment-v1.yaml](01-rolling-update/deployment-v1.yaml), [deployment-v2.yaml](01-rolling-update/deployment-v2.yaml), [service.yaml](01-rolling-update/service.yaml)

Strategy used is `RollingUpdate` with `maxSurge: 1` and `maxUnavailable: 0`.

**Create deployment (v1)**

![](screenshots/01-rolling-v1-apply.png)

![](screenshots/02-rolling-v1-pods.png)

![](screenshots/03-rolling-v1-curl.png)

**Update to v2**

Right after apply, one new v2 pod comes up while all 4 v1 pods are still running (maxSurge 1).

![](screenshots/04-rolling-v2-apply.png)

![](screenshots/05-rolling-status.png)

**Verify old and new pods**

![](screenshots/06-rolling-v2-pods.png)

![](screenshots/07-rolling-v2-curl.png)

Old ReplicaSet is scaled to 0 and kept for rollback:

![](screenshots/08-rolling-history.png)

**Observation:** pods were replaced one by one, there were always 4 ready pods so no downtime.

## 02. Blue-Green Deployment

YAML: [deployment-blue.yaml](02-blue-green/deployment-blue.yaml), [deployment-green.yaml](02-blue-green/deployment-green.yaml), [service-blue.yaml](02-blue-green/service-blue.yaml), [service-green.yaml](02-blue-green/service-green.yaml)

**Create blue and green versions**

![](screenshots/09-bg-apply.png)

![](screenshots/10-bg-pods.png)

**Service points to blue**

![](screenshots/11-bg-blue-active.png)

**Switch traffic to green**

![](screenshots/12-bg-switch-green.png)

![](screenshots/13-bg-green-active.png)

**Rollback to blue** (just apply the blue service again)

![](screenshots/14-bg-rollback-blue.png)

**Observation:** both versions run fully at the same time. Only the service selector (`slot: blue` / `slot: green`) is changed so the switch and rollback are instant. Downside is it needs double resources.

## 03. Canary Deployment

YAML: [deployment-stable.yaml](03-canary/deployment-stable.yaml), [deployment-canary.yaml](03-canary/deployment-canary.yaml), [service.yaml](03-canary/service.yaml)

**Deploy stable version (9 replicas)**

![](screenshots/15-canary-stable.png)

**Deploy canary version (1 replica)**

![](screenshots/16-canary-apply.png)

![](screenshots/17-canary-pods.png)

**Verify traffic split** - sent 30 requests to the service

![](screenshots/18-canary-traffic.png)

**Observation:** both deployments have label `app: myapp-canary` so the one service sends traffic to all 10 pods. 1 out of 10 pods is canary so around 10% of requests go to v2 (I got 2 out of 30). To increase canary traffic, scale canary up and stable down.

## 04. Recreate Deployment

YAML: [deployment-v1.yaml](04-recreate/deployment-v1.yaml), [deployment-v2.yaml](04-recreate/deployment-v2.yaml), [service.yaml](04-recreate/service.yaml)

**Deploy v1**

![](screenshots/19-recreate-v1.png)

![](screenshots/20-recreate-v1-pods.png)

**Update to v2** (watch was running in another terminal)

![](screenshots/21-recreate-v2-apply.png)

![](screenshots/22-recreate-watch.png)

![](screenshots/23-recreate-v2-curl.png)

**Observation:** all 3 v1 pods went to Terminating/Completed first and only after that the v2 pods were created. So there is a small downtime with Recreate.

# Task 2: Pod Lifecycle

YAML files are in [pod-lifecycle/](pod-lifecycle/). For each file I applied it, checked status with `kubectl get pod`, and checked details with `kubectl describe` / `kubectl logs`.

### 01-running.yaml

![](screenshots/lifecycle/01-running.png)

![](screenshots/lifecycle/01-running-details.png)

Normal nginx pod. Scheduled -> Pulled -> Created -> Started, phase is Running.

### 02-pending.yaml

![](screenshots/lifecycle/02-pending.png)

![](screenshots/lifecycle/02-pending-details.png)

Pod requests 9Gi memory and no node has that much, so scheduler can't place it. It stays Pending with `FailedScheduling` event.

### 03-succeeded.yaml

![](screenshots/lifecycle/03-succeeded.png)

![](screenshots/lifecycle/03-succeeded-details.png)

Container exits with code 0 and `restartPolicy: Never`, so status is Completed (phase Succeeded).

### 04-failed.yaml

![](screenshots/lifecycle/04-failed.png)

![](screenshots/lifecycle/04-failed-details.png)

Container exits with code 1 and is not restarted, so status is Error (phase Failed).

### 05-crashloopbackoff.yaml

![](screenshots/lifecycle/05-crashloop.png)

![](screenshots/lifecycle/05-crashloop-details.png)

Default restartPolicy is Always. App keeps crashing, kubelet restarts it with increasing delay, which shows as CrashLoopBackOff and restart count keeps going up.

### 06-imagepullbackoff.yaml

![](screenshots/lifecycle/06-imagepull.png)

![](screenshots/lifecycle/06-imagepull-details.png)

Image name does not exist. First it shows ErrImagePull, then ImagePullBackOff while kubelet waits before retrying.

### 07-readiness.yaml

![](screenshots/lifecycle/07-readiness.png)

![](screenshots/lifecycle/07-readiness-details.png)

Pod is Running but `0/1` ready at first because readiness probe starts after 5s. Once the HTTP probe passes it becomes `1/1`. Only ready pods get traffic from a service.

### 08-liveness.yaml

![](screenshots/lifecycle/08-liveness.png)

![](screenshots/lifecycle/08-liveness-details.png)

After 20s the app deletes `/tmp/healthy`, liveness probe fails 2 times and kubelet kills and restarts the container. Restart count became 1.

### 09-startup.yaml

![](screenshots/lifecycle/09-startup.png)

![](screenshots/lifecycle/09-startup-details.png)

App takes 30s to start. Startup probe fails a few times (allowed up to 10 x 5s) and the pod is `0/1` until `/tmp/started` exists. Then it becomes `1/1` without any restart.

### 10-init-container.yaml

![](screenshots/lifecycle/10-init.png)

![](screenshots/lifecycle/10-init-details.png)

Status shows `Init:0/1` while the init container runs. After it completes (exit 0), the main nginx container starts.

### 11-multi-container.yaml

![](screenshots/lifecycle/11-multi.png)

![](screenshots/lifecycle/11-multi-details.png)

Two containers (nginx + busybox sidecar) in one pod, so READY shows `2/2`. Logs need `-c` to pick the container.

### 12-termination.yaml

![](screenshots/lifecycle/12-termination.png)

![](screenshots/lifecycle/12-termination-details.png)

On delete, kubelet sends SIGTERM. The app traps it, does 10s cleanup and exits. Delete took about 10s, which is within `terminationGracePeriodSeconds: 20`, so no SIGKILL was needed.
