# Solution — Readiness Probe Failure

## Step 1 — Check Deployment

```bash
kubectl get deployment catalog-api
```

You may see:

```text
NAME          READY   UP-TO-DATE   AVAILABLE
catalog-api   0/4     4            0
```

---

## Step 2 — Check Pods

```bash
kubectl get pods
```

The Pods may show:

```text
NAME                         READY   STATUS
catalog-api-xxxxx            0/1     Running
catalog-api-yyyyy            0/1     Running
catalog-api-zzzzz            0/1     Running
catalog-api-aaaaa            0/1     Running
```

This is an important production distinction:

```text
STATUS = Running
```

does NOT mean:

```text
READY = 1/1
```

---

# Step 3 — Describe a Pod

```bash
kubectl describe pod <pod-name>
```

Look at Events.

You should find readiness probe failures similar to:

```text
Readiness probe failed
connection refused
```

---

# Step 4 — Understand the Probe

The manifest contains:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
```

But the container is running:

```text
nginx
```

and listening on:

```text
80
```

not:

```text
8080
```

There is also no `/health` endpoint on port `8080`.

---

# Root Cause

The readiness probe is checking the wrong endpoint:

```text
Pod
 |
 +-- nginx listens on 80
 |
 +-- readinessProbe checks 8080
                         X
```

Therefore Kubernetes marks the Pod:

```text
Ready = false
```

The Deployment sees:

```text
Available replicas = 0
```

even though the containers themselves are running.

---

# Step 5 — Fix the Probe

Change:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
```

For this lab, use an endpoint that exists on the NGINX container:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
```

---

# Corrected Manifest

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: catalog-api

spec:
  replicas: 4

  strategy:
    type: RollingUpdate

  selector:
    matchLabels:
      app: catalog

  template:
    metadata:
      labels:
        app: catalog

    spec:
      containers:
        - name: catalog
          image: nginx:latest

          ports:
            - containerPort: 80

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
```

Apply:

```bash
kubectl apply -f broken.yaml
```

---

# Step 6 — Monitor

```bash
kubectl get pods -w
```

Then:

```bash
kubectl rollout status deployment catalog-api
```

---

# Step 7 — Verify

```bash
kubectl get deployment catalog-api
```

Expected:

```text
NAME          READY   UP-TO-DATE   AVAILABLE
catalog-api   4/4     4            4
```

---

# Production Lesson

Remember:

```text
Running ≠ Ready
```

A container can be running while Kubernetes considers it unavailable.

The readiness probe controls whether the Pod is considered ready to receive traffic.

Typical causes of readiness failures:

* Wrong port
* Wrong HTTP path
* Application startup taking longer
* Dependency unavailable
* Incorrect probe command
* TLS mismatch
* Authentication requirements
* Application actually unhealthy

---

# Certification Troubleshooting Pattern

```text
Deployment unavailable
        ↓
kubectl get pods
        ↓
Running but 0/1 Ready?
        ↓
kubectl describe pod
        ↓
Inspect readinessProbe
        ↓
Check application port/path
        ↓
Fix probe
        ↓
Verify Ready
```
