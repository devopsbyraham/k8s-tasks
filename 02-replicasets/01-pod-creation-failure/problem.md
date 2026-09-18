# ReplicaSet Lab 01 — Pod Creation Failure

## Production Scenario

You are on-call for an e-commerce platform.

The `catalog-api` team deployed a new ReplicaSet expecting **3 Pods** to run.

However, only the ReplicaSet object exists and no application Pods are becoming Ready.

The application team reports:

> "The ReplicaSet was deployed successfully, but the expected Pods are not running."

Your task is to identify the root cause without deleting the ReplicaSet.

---

## Objective

Fix the ReplicaSet so that:

```text
Desired: 3
Current: 3
Ready:   3
```

The ReplicaSet name must remain:

```text
catalog-api-rs
```

---

## Step 1 — Deploy

```bash
kubectl apply -f broken.yaml
```

---

## Step 2 — Investigate

Start with:

```bash
kubectl get rs
```

Then:

```bash
kubectl get pods
```

Check the ReplicaSet:

```bash
kubectl describe rs catalog-api-rs
```

Check events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

## Rules

Do not:

* Delete the ReplicaSet.
* Create Pods manually.
* Replace the ReplicaSet with a Deployment.
* Increase the replica count to hide the problem.

Find and fix the actual configuration problem.

---

## Expected Result

```text
kubectl get rs
```

should show:

```text
NAME             DESIRED   CURRENT   READY
catalog-api-rs   3         3         3
```

---

## Certification Skills

Practice:

* ReplicaSet troubleshooting
* Pod template validation
* ReplicaSet events
* Image configuration
* Controller behavior
* `kubectl describe`
