# Lab 03 — Pod Stuck in Pending

## Scenario

A developer deployed a reporting application.

The Pod has been created successfully, but Kubernetes has not scheduled it onto any node.

## Objective

Troubleshoot why the Pod remains:

```text
Pending
```

Identify the scheduling problem and fix it.

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

The Pod remains:

```text
Pending
```

Your task is to determine why.

## Useful Commands

Start with:

```bash
kubectl get pod report-api
```

Then:

```bash
kubectl describe pod report-api
```

Also inspect:

```bash
kubectl get nodes --show-labels
```

## Restrictions

* Do not delete the Pod.
* Do not add additional worker nodes.
* Do not remove the application's node-selection requirement blindly.
* Find the scheduling mismatch.

## Expected Result

The Pod should become:

```text
Running
```

## Certification Skill

Practice:

* Pod scheduling
* Node labels
* `nodeSelector`
* Pending Pods
* Scheduler events
