# Solution — CrashLoopBackOff

## Step 1 — Check the Pod

```bash
kubectl get pod payment-api
```

You may see:

```text
NAME          READY   STATUS             RESTARTS
payment-api   0/1     CrashLoopBackOff   ...
```

## Step 2 — Check the Logs

```bash
kubectl logs payment-api
```

You should see:

```text
Starting payment API
Connecting to payment database
```

The container is terminating after starting.

## Step 3 — Check the Previous Container

Because Kubernetes is restarting the container, this command is useful:

```bash
kubectl logs payment-api --previous
```

## Step 4 — Inspect the Pod

```bash
kubectl describe pod payment-api
```

Look at:

```text
Last State
Exit Code
Reason
Restart Count
```

The container exits with:

```text
Exit Code: 1
```

## Root Cause

The container's command explicitly executes:

```bash
exit 1
```

Exit code `1` indicates that the process terminated unsuccessfully.

Kubernetes restarts the container according to the Pod's restart behavior, eventually producing:

```text
CrashLoopBackOff
```

## Step 5 — Fix the Application Command

Change:

```yaml
exit 1
```

to a long-running process:

```yaml
sleep 3600
```

Corrected manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment-api
spec:
  containers:
    - name: payment
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          echo "Starting payment API"
          echo "Connecting to payment database"
          echo "Payment API started successfully"
          sleep 3600
```

Apply:

```bash
kubectl apply -f broken.yaml
```

## Step 6 — Verify

```bash
kubectl get pod payment-api
```

Expected:

```text
NAME          READY   STATUS    RESTARTS
payment-api   1/1     Running   0
```

## Key Takeaway

When a Pod is in:

```text
CrashLoopBackOff
```

follow this sequence:

```text
kubectl get pod
        ↓
kubectl describe pod
        ↓
kubectl logs
        ↓
kubectl logs --previous
        ↓
Check Exit Code
        ↓
Identify application failure
        ↓
Fix
        ↓
Verify
```

### Certification Tip

Do not assume `CrashLoopBackOff` itself is the root cause.

It is a symptom.

The actual cause may be:

* Application error
* Wrong command
* Missing configuration
* Missing Secret
* Missing ConfigMap
* Permission problem
* Dependency failure
* Failed startup process
