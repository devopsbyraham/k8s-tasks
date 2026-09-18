# Solution — Selector / Label Mismatch

## Step 1 — Check ReplicaSet

```bash
kubectl get rs
```

You will observe:

```text
NAME            DESIRED   CURRENT   READY
orders-api-rs   4         0         0
```

The ReplicaSet wants four Pods but has created none.

---

## Step 2 — Describe the ReplicaSet

```bash
kubectl describe rs orders-api-rs
```

Inspect the:

```text
Selector
```

It contains:

```text
app=orders-api
```

---

## Step 3 — Inspect the Manifest

ReplicaSet selector:

```yaml
selector:
  matchLabels:
    app: orders-api
```

But the Pod template contains:

```yaml
labels:
  app: order-api
```

Notice the difference:

```text
orders-api
     vs
order-api
```

The label does not match.

---

# Root Cause

The ReplicaSet selector must match the labels on the Pod template.

Current configuration:

```text
ReplicaSet selector
        |
        | app=orders-api
        X
        |
Pod template
app=order-api
```

Because the selector and template labels do not match, Kubernetes cannot create a valid ReplicaSet configuration.

---

# Step 4 — Fix the Label

Change:

```yaml
labels:
  app: order-api
```

to:

```yaml
labels:
  app: orders-api
```

Corrected manifest:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: orders-api-rs

spec:
  replicas: 4

  selector:
    matchLabels:
      app: orders-api

  template:
    metadata:
      labels:
        app: orders-api

    spec:
      containers:
        - name: orders
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f broken.yaml
```

---

# Step 5 — Verify

```bash
kubectl get rs
```

Expected:

```text
NAME            DESIRED   CURRENT   READY
orders-api-rs   4         4         4
```

Check Pods:

```bash
kubectl get pods --show-labels
```

All four Pods should contain:

```text
app=orders-api
```

---

# Important Certification Concept

A ReplicaSet uses:

```yaml
spec.selector
```

to determine which Pods belong to it.

The Pod template defines:

```yaml
spec.template.metadata.labels
```

Those labels must satisfy the selector.

Think:

```text
SELECTOR
   ↓
"Find Pods with this label"
   ↓
POD LABEL
```

For this lab:

```text
selector:
  app=orders-api

template:
  app=orders-api
```

They match.

---

# Production Debugging Pattern

If:

```text
DESIRED = 4
CURRENT = 0
```

check:

```bash
kubectl describe rs <name>
kubectl get rs <name> -o yaml
```

Then verify:

```text
selector
    ↕
template labels
```

Never assume that a ReplicaSet with zero Pods means the cluster is out of capacity.
