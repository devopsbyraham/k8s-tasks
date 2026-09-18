# Lab 02 — CrashLoopBackOff

## Scenario

The development team reports that the `payment-api` Pod starts but repeatedly becomes unavailable.

The Pod exists, but its container keeps restarting.

## Objective

Troubleshoot the Pod and identify why the container is continuously restarting.

Fix the Pod so that it remains in:

```text
Running
```

## Starting Environment

Apply:

```bash
kubectl apply -f broken.yaml
```

Check:

```bash
kubectl get pods
```

## Problem

You observe:

```text
CrashLoopBackOff
```

Find the root cause.

## Useful Commands

```bash
kubectl get pod payment-api
kubectl describe pod payment-api
kubectl logs payment-api
kubectl logs payment-api --previous
```

## Restrictions

* Do not delete the Pod.
* Do not change the Pod name.
* Do not change the container image.
* Identify the actual application failure.

## Expected Result

The Pod should remain:

```text
1/1 Running
```

## Certification Skill

Practice:

* CrashLoopBackOff
* Container exit codes
* Container logs
* Previous container logs
* Command configuration
