# Lab 02 — Readiness Probe Failure

## Production Scenario

The `catalog-api` Deployment was successfully updated.

All Pods show:

```text
Running
```

However, the application team reports that traffic is not reaching the new Pods.

The rollout is also not completing.

Your monitoring system reports:

```text
Available replicas < desired replicas
```

Your task is to determine why Kubernetes considers the Pods unavailable.

---

## Objective

Troubleshoot the Deployment and make all replicas become:

```text
Ready
```

---

## Deploy

```bash
kubectl apply -f broken.yaml
```

---

## Initial Investigation

```bash
kubectl get deployment
kubectl get pods
kubectl get rs
```

Then:

```bash
kubectl rollout status deployment catalog-api
```

---

## Important Question

The Pods are:

```text
Running
```

but why are they not:

```text
Ready
```

---

## Useful Commands

```bash
kubectl describe deployment catalog-api
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

Check Pod conditions:

```bash
kubectl get pods
```

---

## Restrictions

Do not:

* Remove the readiness probe without understanding the problem.
* Delete the Deployment.
* Delete all Pods.
* Scale the Deployment to zero.

Fix the actual configuration.

---

## Expected Result

```text
NAME          READY   UP-TO-DATE   AVAILABLE
catalog-api   4/4     4            4
```

---

## Certification Skills

Practice:

* Readiness probes
* Pod readiness
* Deployment availability
* Rollout troubleshooting
* Application health checks
