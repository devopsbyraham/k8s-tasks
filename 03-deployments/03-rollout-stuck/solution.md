# Solution — Rollout Stuck

## Step 1 — Check Deployment

```bash
kubectl get deployment frontend
```

You may see:

```text
NAME       READY   UP-TO-DATE   AVAILABLE
frontend   4/5     1            4
```

The Deployment is not fully available.

---

# Step 2 — Check Rollout

```bash
kubectl rollout status deployment frontend
```

The rollout does not complete.

---

# Step 3 — Inspect ReplicaSets

```bash
kubectl get rs
```

You may see:

```text
NAME                  DESIRED   CURRENT   READY
frontend-old          4         4         4
frontend-new          1         1         0
```

The old ReplicaSet has four healthy Pods.

The new ReplicaSet has one Pod, but it is not Ready.

---

# Step 4 — Inspect the New Pod

```bash
kubectl describe pod <new-pod-name>
```

Look at Events.

You should find:

```text
Readiness probe failed
HTTP probe failed with statuscode: 404
```

---

# Step 5 — Identify the Probe Problem

The manifest contains:

```yaml
readinessProbe:
  httpGet:
    path: /does-not-exist
    port: 80
```

NGINX does not serve this path.

Therefore:

```text
Container
    ↓
Running
    ↓
Readiness probe
    ↓
HTTP 404
    ↓
Pod NOT READY
```

---

# Step 6 — Understand Why the Rollout Stops

The Deployment uses:

```yaml
maxUnavailable: 0
maxSurge: 1
```

This means Kubernetes must maintain all existing available replicas while updating.

Conceptually:

```text
5 replicas required
        |
        +---- old healthy Pods = 4
        |
        +---- new Pod = not Ready
```

Because the new Pod is not becoming Ready, the Deployment cannot safely continue reducing the old ReplicaSet.

The rollout waits.

---

# Root Cause

The immediate root cause is the incorrect readiness probe:

```yaml
path: /does-not-exist
```

The rollout strategy then exposes the problem because:

```yaml
maxUnavailable: 0
```

prevents Kubernetes from intentionally reducing available capacity during the rollout.

---

# Step 7 — Fix the Readiness Probe

Change:

```yaml
path: /does-not-exist
```

to:

```yaml
path: /
```

Corrected section:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 5
  failureThreshold: 2
```

Apply:

```bash
kubectl apply -f broken.yaml
```

---

# Step 8 — Watch the Rollout

```bash
kubectl rollout status deployment frontend
```

Then:

```bash
kubectl get rs
```

You should see the new ReplicaSet gradually become healthy.

---

# Step 9 — Verify

```bash
kubectl get deployment frontend
```

Expected:

```text
NAME       READY   UP-TO-DATE   AVAILABLE
frontend   5/5     5            5
```

Check Pods:

```bash
kubectl get pods
```

All five Pods should be:

```text
1/1 Running
```

---

# Production Mental Model

During a Deployment rollout, think about three objects:

```text
Deployment
     |
     +----------------+
     |                |
     v                v
Old ReplicaSet     New ReplicaSet
     |                |
     v                v
Old Pods           New Pods
```

The Deployment controls the ReplicaSets.

The ReplicaSets control the Pods.

Pod readiness determines whether those Pods are available.

---

# Golden Troubleshooting Flow

When a Deployment rollout is stuck:

```text
kubectl rollout status
          ↓
kubectl get deployment
          ↓
kubectl get rs
          ↓
Compare OLD vs NEW ReplicaSet
          ↓
kubectl get pods
          ↓
kubectl describe pod
          ↓
Check readiness/liveness/events
          ↓
Fix root cause
          ↓
Verify rollout
```

---

# Important Certification Concept

Understand the difference:

```text
maxUnavailable
```

controls how many Pods can be unavailable during the update.

```text
maxSurge
```

controls how many additional Pods can temporarily exist above the desired replica count.

For this lab:

```text
replicas = 5
maxUnavailable = 0
maxSurge = 1
```

Kubernetes can temporarily create:

```text
6 Pods
```

but should maintain:

```text
5 available Pods
```

through the rollout.

The broken readiness probe prevents the new Pod from becoming available, so the rollout cannot progress normally.
