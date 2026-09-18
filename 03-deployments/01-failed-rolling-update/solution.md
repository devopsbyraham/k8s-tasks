# Solution — Failed Rolling Update

## Step 1 — Check Deployment

```bash
kubectl get deployment checkout-api
```

You may see:

```text
NAME           READY   UP-TO-DATE   AVAILABLE
checkout-api   3/4     1            3
```

This indicates that the rollout is not healthy.

---

## Step 2 — Check Rollout Status

```bash
kubectl rollout status deployment checkout-api
```

The command will wait because the new version has not become available.

---

## Step 3 — Check ReplicaSets

```bash
kubectl get rs
```

You should see multiple ReplicaSets.

For example:

```text
NAME                       DESIRED   CURRENT   READY
checkout-api-7f8d9xxxx     3         3         3
checkout-api-6c4b7xxxx     1         1         0
```

The older ReplicaSet is healthy.

The new ReplicaSet has Pods that are not Ready.

---

## Step 4 — Check Pods

```bash
kubectl get pods
```

You should find a Pod with:

```text
ImagePullBackOff
```

or:

```text
ErrImagePull
```

---

## Step 5 — Inspect the Failed Pod

```bash
kubectl describe pod <new-pod-name>
```

Look at the Events section.

The Deployment is attempting to pull:

```text
nginx:1.26-does-not-exist
```

---

# Root Cause

The new Deployment version references an invalid image:

```yaml
image: nginx:1.26-does-not-exist
```

The old version is healthy.

The new ReplicaSet cannot start its Pods.

Therefore:

```text
Old ReplicaSet
      ↓
Healthy

New ReplicaSet
      ↓
ImagePullBackOff
      ↓
Rollout cannot complete
```

---

# Production Decision

At this point, there are two possible approaches.

### Option 1 — Fix the image

If version `1.26` is actually valid in your real registry, correct the image reference.

For this lab, the requested image is intentionally invalid.

### Option 2 — Roll Back

Because production is already impacted, use the Deployment's rollback mechanism.

First inspect history:

```bash
kubectl rollout history deployment checkout-api
```

Then roll back:

```bash
kubectl rollout undo deployment checkout-api
```

---

# Step 6 — Monitor Rollback

```bash
kubectl rollout status deployment checkout-api
```

Then:

```bash
kubectl get pods
kubectl get rs
```

---

# Step 7 — Verify

```bash
kubectl get deployment checkout-api
```

Expected:

```text
NAME           READY   UP-TO-DATE   AVAILABLE
checkout-api   4/4     4            4
```

---

# Certification Pattern

When a Deployment rollout is stuck:

```text
kubectl rollout status
        ↓
kubectl get rs
        ↓
kubectl get pods
        ↓
kubectl describe pod
        ↓
Identify failed new ReplicaSet
        ↓
Fix or rollback
        ↓
kubectl rollout status
```

## Important Production Lesson

Never blindly execute:

```bash
kubectl delete pods --all
```

during a production incident.

First determine whether:

* The old ReplicaSet is healthy.
* The new ReplicaSet is healthy.
* The new image can be pulled.
* The rollout is progressing.
* A rollback is appropriate.

A Deployment gives you controlled rollout and rollback capabilities.
