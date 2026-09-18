# Lab 01 — ImagePullBackOff

## Scenario

The application team deployed an NGINX Pod, but the Pod is not starting.

Your task is to troubleshoot the Pod and identify why Kubernetes cannot start the container.

## Objective

Fix the Pod so that:

* The Pod reaches `Running` state.
* The container starts successfully.
* The Pod name must remain `nginx-app`.
* Do not delete the namespace.

## Starting Environment

Apply the provided manifest:

```bash
kubectl apply -f broken.yaml
```

Check the Pod:

```bash
kubectl get pods
```

## Problem

The Pod is showing:

```text
ImagePullBackOff
```

Diagnose the problem using Kubernetes commands.

## Rules

Do not immediately replace the entire manifest.

Use troubleshooting commands such as:

```bash
kubectl get pod nginx-app
kubectl describe pod nginx-app
kubectl get events --sort-by=.lastTimestamp
```

## Expected Result

After fixing the issue:

```text
NAME         READY   STATUS    RESTARTS   AGE
nginx-app    1/1     Running   0          ...
```

## Certification Skill

You should be able to troubleshoot:

* ImagePullBackOff
* ErrImagePull
* Container image configuration
* Pod events
* `kubectl describe`
