Lab 01 — Failed Rolling Update
Production Scenario

You are the on-call DevOps engineer for an e-commerce platform.

The checkout-api application was working correctly with version 1.25.

A new deployment was initiated to release version 1.26.

Shortly after the deployment, the application team reports:

"The new version is not becoming Ready and the rollout appears to be stuck."

Your task is to investigate the rollout and identify the problem.

Objective

Restore the Deployment to a healthy state.

The Deployment must eventually have:

Desired: 4
Current: 4
Ready:   4

The application must run successfully.

Deploy
kubectl apply -f broken.yaml

Check:

kubectl get deployment
kubectl get pods
kubectl get rs
Investigation

Use:

kubectl rollout status deployment checkout-api

Then:

kubectl rollout history deployment checkout-api

Inspect the Deployment:

kubectl describe deployment checkout-api

Inspect ReplicaSets:

kubectl get rs

Inspect Pods:

kubectl get pods
Challenge

Determine:

Which version is failing?
Which ReplicaSet belongs to the failed rollout?
Why are the new Pods not becoming Ready?
How can you restore the application safely?
Restrictions

Do not:

Delete the Deployment.
Delete all Pods.
Scale the Deployment to zero.
Manually create replacement Pods.

Use the Deployment's rollout capabilities.

Expected Result
kubectl get deployment

should eventually show:

NAME           READY   UP-TO-DATE   AVAILABLE
checkout-api   4/4     4            4
Certification Skills

Practice:

Deployments
Rolling updates
ReplicaSets
Rollout status
Rollout history
Rollback
Production troubleshooting
