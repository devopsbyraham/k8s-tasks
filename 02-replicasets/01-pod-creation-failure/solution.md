# Solution — Pod Creation Failure

## Step 1 — Check the ReplicaSet

```bash
kubectl get rs
```

You may see:

```text
NAME             DESIRED   CURRENT   READY
catalog-api-rs   3         3         0
```

This is an important clue.

The ReplicaSet has created the desired number of Pods, but they are not Ready.

---

## Step 2 — Check Pods

```bash
kubectl get pods
```

You should see something similar to:

```text
catalog-api-rs-xxxxx   0/1   ImagePullBackOff
catalog-api-rs-yyyyy   0/1   ImagePullBackOff
catalog-api-rs-zzzzz   0/1   ImagePullBackOff
```

---

## Step 3 — Inspect the ReplicaSet

```bash
kubectl describe rs catalog-api-rs
```

Look at:

```text
Events
```

The ReplicaSet itself is working.

It successfully created the Pods.

The failure is occurring while the Pods are starting.

---

## Step 4 — Inspect a Pod

```bash
kubectl describe pod <pod-name>
```

Look at the container image:

```text
nginx:prod-v99
```

The image/tag does not exist.

---

# Root Cause

The ReplicaSet controller is functioning correctly.

The problem is the invalid container image:

```yaml
image: nginx:prod-v99
```

This causes:

```text
ErrImagePull
        ↓
ImagePullBackOff
```

The ReplicaSet therefore has:

```text
Desired = 3
Current = 3
Ready   = 0
```

---

# Step 5 — Fix the Image

Change:

```yaml
image: nginx:prod-v99
```

to:

```yaml
image: nginx:latest
```

Corrected manifest:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: catalog-api-rs
spec:
  replicas: 3

  selector:
    matchLabels:
      app: catalog-api

  template:
    metadata:
      labels:
        app: catalog-api

    spec:
      containers:
        - name: catalog-api
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f broken.yaml
```

---

# Step 6 — Verify

```bash
kubectl get pods
```

Then:

```bash
kubectl get rs
```

Expected:

```text
NAME             DESIRED   CURRENT   READY
catalog-api-rs   3         3         3
```

---

# Production Troubleshooting Pattern

When:

```text
DESIRED = 3
CURRENT = 3
READY   = 0
```

the ReplicaSet controller may already be doing its job.

Do not immediately troubleshoot the ReplicaSet selector.

Inspect the Pods:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
```

---

# Key Takeaway

A ReplicaSet is responsible for maintaining the desired number of Pods.

It does **not** guarantee that the application inside those Pods is healthy.

```text
ReplicaSet
    ↓
Creates Pods
    ↓
Pods fail to start
    ↓
READY remains 0
```

Always distinguish:

```text
ReplicaSet problem
        vs
Pod/container problem
```
