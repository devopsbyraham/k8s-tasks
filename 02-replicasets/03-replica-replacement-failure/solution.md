# Solution — Replica Replacement Failure

## Step 1 — Check the ReplicaSet

```bash
kubectl get rs
```

You will notice that the desired number is five, but the current number is not five.

Example:

```text
NAME          DESIRED   CURRENT   READY
frontend-rs   5         0         0
```

---

# Step 2 — Inspect the ReplicaSet

```bash
kubectl describe rs frontend-rs
```

Look at:

```text
Selector
```

The ReplicaSet expects:

```text
app=frontend
```

---

# Step 3 — Inspect the Pod Template

Run:

```bash
kubectl get rs frontend-rs -o yaml
```

Look at:

```yaml
spec:
  template:
    metadata:
      labels:
```

You will find:

```yaml
labels:
  app: frontend-v2
```

Again, there is a mismatch:

```text
ReplicaSet selector:

app=frontend


Pod template:

app=frontend-v2
```

---

# Root Cause

The ReplicaSet controller uses the selector to identify the Pods it manages.

The selector says:

```yaml
app: frontend
```

but the Pod template creates Pods with:

```yaml
app: frontend-v2
```

Therefore, the Pod template does not satisfy the ReplicaSet selector.

The controller cannot maintain the expected ReplicaSet membership.

---

# Step 4 — Fix the Template Label

Change:

```yaml
labels:
  app: frontend-v2
```

to:

```yaml
labels:
  app: frontend
```

Corrected manifest:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend-rs

spec:
  replicas: 5

  selector:
    matchLabels:
      app: frontend

  template:
    metadata:
      labels:
        app: frontend

    spec:
      containers:
        - name: frontend
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f broken.yaml
```

---

# Step 5 — Verify Reconciliation

Watch the ReplicaSet:

```bash
kubectl get rs -w
```

In another terminal:

```bash
kubectl get pods -w
```

Eventually:

```text
NAME          DESIRED   CURRENT   READY
frontend-rs   5         5         5
```

---

# Step 6 — Verify Selector

Run:

```bash
kubectl get pods -l app=frontend
```

You should see five Pods.

Then:

```bash
kubectl get pods --show-labels
```

The Pods should have:

```text
app=frontend
```

---

# Production-Level Lesson

A ReplicaSet is a controller implementing a reconciliation loop.

Conceptually:

```text
Desired State
      |
      v
ReplicaSet
      |
      v
Observe Cluster
      |
      v
Compare
      |
      +---- Desired != Current
      |
      v
Take Corrective Action
      |
      v
Observe Again
```

For example:

```text
Desired = 5
Current = 4

ReplicaSet
    ↓
Create replacement Pod
    ↓
Current = 5
```

If the ReplicaSet's selector/template relationship is incorrectly configured, this reconciliation behavior cannot produce the intended workload state.

---

# Certification Troubleshooting Sequence

When a ReplicaSet is not maintaining replicas:

```bash
kubectl get rs
```

Then:

```bash
kubectl describe rs <rs-name>
```

Then:

```bash
kubectl get pods --show-labels
```

Then inspect:

```bash
kubectl get rs <rs-name> -o yaml
```

Compare:

```text
spec.selector.matchLabels
             ↕
spec.template.metadata.labels
```

---

# Golden Rule

For ReplicaSets, remember:

```text
SELECTOR
   =
POD LABEL
```

If they do not match correctly, the ReplicaSet cannot manage the intended Pods.

Also remember:

```text
Desired ≠ Current
```

is a signal to investigate the controller's reconciliation process rather than manually creating Pods.
