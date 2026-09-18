# Lab 03 — Replica Replacement Failure

## Production Scenario

You are troubleshooting a production `frontend` workload.

The ReplicaSet is configured for:

```text
5 replicas
```

During an incident, one of the frontend Pods becomes unavailable.

The application team reports:

> "We lost one frontend Pod, but Kubernetes did not restore the expected replica count."

Your task is to determine what is happening.

---

## Objective

Ensure the ReplicaSet maintains:

```text
Desired = 5
Current = 5
Ready   = 5
```

---

## Deploy

```bash
kubectl apply -f broken.yaml
```

Check:

```bash
kubectl get rs
kubectl get pods -o wide
```

---

## Investigation

Use:

```bash
kubectl describe rs frontend-rs
```

Check Pod ownership:

```bash
kubectl get pods --show-labels
```

Inspect ReplicaSet YAML:

```bash
kubectl get rs frontend-rs -o yaml
```

You may also use:

```bash
kubectl get pods -l app=frontend
```

---

## Rules

Do not:

* Manually create replacement Pods.
* Increase replicas above 5.
* Delete the ReplicaSet.
* Replace it with a Deployment.

The ReplicaSet must be able to maintain its desired state.

---

## Expected Result

```text
NAME          DESIRED   CURRENT   READY
frontend-rs   5         5         5
```

---

## Certification Skills

Practice:

* ReplicaSet reconciliation
* Desired vs current state
* Selectors
* Pod ownership
* Labels
* Controller troubleshooting
