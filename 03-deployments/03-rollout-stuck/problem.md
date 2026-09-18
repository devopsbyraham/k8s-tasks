# Lab 03 — Rollout Stuck Due to Availability Constraints

## Production Scenario

You are deploying a new version of the `frontend` application.

The Deployment uses:

```text
replicas: 5
maxUnavailable: 0
maxSurge: 1
```

The deployment team reports:

> "The rollout has been running for several minutes and never completes."

The old application version is healthy.

The new version starts, but the rollout does not progress.

Your task is to identify why Kubernetes cannot complete the rollout.

---

## Objective

Identify the rollout blocker and restore the Deployment to a healthy state.

The final Deployment must have:

```text
5/5 Ready
```

---

## Deploy

```bash
kubectl apply -f broken.yaml
```

---

## Investigation

Start with:

```bash
kubectl get deployment frontend
```

Then:

```bash
kubectl rollout status deployment frontend
```

Check ReplicaSets:

```bash
kubectl get rs
```

Check Pods:

```bash
kubectl get pods
```

Inspect the Deployment:

```bash
kubectl describe deployment frontend
```

Inspect the new Pod:

```bash
kubectl describe pod <new-pod>
```

---

## Key Question

Why can Kubernetes create the new Pod but not complete the rollout?

Think about:

* `maxUnavailable`
* `maxSurge`
* Pod readiness
* Old ReplicaSet
* New ReplicaSet

---

## Restrictions

Do not:

* Delete the Deployment.
* Delete all Pods.
* Scale down the application manually.
* Disable the rolling update strategy.

---

## Expected Result

```text
NAME       READY   UP-TO-DATE   AVAILABLE
frontend   5/5     5            5
```

---

## Certification Skills

Practice:

* RollingUpdate strategy
* maxUnavailable
* maxSurge
* Deployment conditions
* ReplicaSets
* Readiness
* Rollout troubleshooting
