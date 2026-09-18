# Lab 02 — ReplicaSet Selector / Label Mismatch

## Production Scenario

The `orders-api` team reports that their ReplicaSet is not maintaining the expected number of replicas.

They expect **4 Pods**, but the ReplicaSet reports:

```text
DESIRED = 4
CURRENT = 0
READY   = 0
```

The cluster has sufficient capacity.

Your task is to troubleshoot the ReplicaSet configuration.

---

## Objective

Make the ReplicaSet create and maintain exactly:

```text
4 Pods
```

Do not manually create Pods.

---

## Deploy

```bash
kubectl apply -f broken.yaml
```

---

## Investigate

Run:

```bash
kubectl get rs
```

Then:

```bash
kubectl describe rs orders-api-rs
```

Inspect the ReplicaSet YAML:

```bash
kubectl get rs orders-api-rs -o yaml
```

Also inspect existing Pods:

```bash
kubectl get pods --show-labels
```

---

## Expected Result

```text
NAME            DESIRED   CURRENT   READY
orders-api-rs   4         4         4
```

---

## Certification Skills

Practice:

* ReplicaSet selectors
* Pod template labels
* Label matching
* Controller ownership
* ReplicaSet troubleshooting
