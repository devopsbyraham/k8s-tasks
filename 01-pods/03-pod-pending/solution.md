# Solution — Pod Stuck in Pending

## Step 1 — Check the Pod

```bash
kubectl get pod report-api
```

You should see:

```text
NAME         READY   STATUS    RESTARTS   AGE
report-api   0/1     Pending   0          ...
```

## Step 2 — Describe the Pod

```bash
kubectl describe pod report-api
```

Check the `Events` section.

You will see a scheduling-related message indicating that no node satisfies:

```text
nodeSelector:
  workload: reporting
```

## Step 3 — Inspect Node Labels

Run:

```bash
kubectl get nodes --show-labels
```

You may see that the available node has labels similar to:

```text
kubernetes.io/hostname=worker-01
```

but does not have:

```text
workload=reporting
```

## Root Cause

The Pod requires:

```yaml
nodeSelector:
  workload: reporting
```

But no available node has that label.

Therefore, the Kubernetes scheduler cannot find a suitable node.

The Pod remains:

```text
Pending
```

## Step 4 — Add the Required Label

First identify the node:

```bash
kubectl get nodes
```

For example:

```text
NAME       STATUS   ROLES
worker-01  Ready    <none>
```

Add the label:

```bash
kubectl label node worker-01 workload=reporting
```

Verify:

```bash
kubectl get nodes --show-labels
```

You should now see:

```text
workload=reporting
```

## Step 5 — Verify the Pod

```bash
kubectl get pod report-api -w
```

The Pod should transition:

```text
Pending
   ↓
ContainerCreating
   ↓
Running
```

Final result:

```text
NAME         READY   STATUS    RESTARTS
report-api   1/1     Running   0
```

## Key Takeaway

For a Pod stuck in:

```text
Pending
```

do not immediately assume the container is broken.

The Pod may not have been scheduled yet.

Use:

```bash
kubectl describe pod <pod-name>
```

and inspect scheduler events.

### Certification Pattern

```text
Pending
   ↓
kubectl describe pod
   ↓
Check Events
   ↓
Check Nodes
   ↓
Check nodeSelector / affinity / taints
   ↓
Fix scheduling constraint
   ↓
Verify Running
```

### Important

A `Pending` Pod can be caused by many things, including:

* Insufficient CPU/memory
* nodeSelector mismatch
* Node affinity
* Taints and tolerations
* Unsatisfied topology constraints
* PVC binding issues
* Scheduler constraints

Always use the events to identify the actual reason.
