# Lab 01 — Failed Rolling Update

## Production Scenario

You are the on-call DevOps engineer for an e-commerce platform.

The `checkout-api` application was working correctly with version `1.25`.

A new deployment was initiated to release version `1.26`.

Shortly after the deployment, the application team reports:

> "The new version is not becoming Ready and the rollout appears to be stuck."

Your task is to investigate the rollout and identify the problem.

---

## Objective

Restore the Deployment to a healthy state.

The Deployment must eventually have:

```text
Desired: 4
Current: 4
Ready:   4
```

The application must run successfully.

---

## Deploy

```bash
kubectl apply -f broken.yaml
```

Check:

```bash
kubectl get deployment
kubectl get pods
kubectl get rs
```

---

## Investigation

Use:

```bash
kubectl rollout status deployment checkout-api
```

Then:

```bash
kubectl rollout history deployment checkout-api
```

Inspect the Deployment:

```bash
kubectl describe deployment checkout-api
```

Inspect ReplicaSets:

```bash
kubectl get rs
```

Inspect Pods:

```bash
kubectl get pods
```

---

## Challenge

Determine:

1. Which version is failing?
2. Which ReplicaSet belongs to the failed rollout?
3. Why are the new Pods not becoming Ready?
4. How can you restore the application safely?

---

## Restrictions

Do not:

* Delete the Deployment.
* Delete all Pods.
* Scale the Deployment to zero.
* Manually create replacement Pods.

Use the Deployment's rollout capabilities.

---

## Expected Result

```text
kubectl get deployment
```

should eventually show:

```text
NAME           READY   UP-TO-DATE   AVAILABLE
checkout-api   4/4     4            4
```

---

## Certification Skills

Practice:

* Deployments
* Rolling updates
* ReplicaSets
* Rollout status
* Rollout history
* Rollback
* Production troubleshooting
