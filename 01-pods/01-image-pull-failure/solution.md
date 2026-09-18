# Solution — ImagePullBackOff

## Step 1 — Check the Pod

```bash
kubectl get pods
```

You should see something similar to:

```text
NAME        READY   STATUS             RESTARTS   AGE
nginx-app   0/1     ImagePullBackOff   0          ...
```

## Step 2 — Describe the Pod

```bash
kubectl describe pod nginx-app
```

Look at the `Events` section.

You will find an error indicating that Kubernetes cannot pull:

```text
nginx:does-not-exist
```

## Root Cause

The image tag does not exist.

The Pod specification contains:

```yaml
image: nginx:does-not-exist
```

## Step 3 — Fix the Image

Edit the manifest:

```yaml
image: nginx:latest
```

Corrected manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

Apply it:

```bash
kubectl apply -f broken.yaml
```

## Step 4 — Verify

```bash
kubectl get pod nginx-app
```

Expected:

```text
NAME        READY   STATUS    RESTARTS   AGE
nginx-app   1/1     Running   0          ...
```

Check the container:

```bash
kubectl describe pod nginx-app
```

## Key Takeaway

When you see:

```text
ImagePullBackOff
```

check:

```bash
kubectl describe pod <pod-name>
```

Then inspect the image name, tag, registry access and Pod events.

### Certification Pattern

```text
ImagePullBackOff
        ↓
kubectl describe pod
        ↓
Check Events
        ↓
Validate image
        ↓
Fix image/tag
        ↓
Verify Running
```
