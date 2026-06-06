# Cluster Bootstrap Repository - App of Apps Pattern with ArgoCD

## Introduction

This repository demonstrates cluster bootstrapping using the ArgoCD App of Apps pattern.

The objective is to manage an entire Kubernetes cluster from Git using a hierarchical GitOps architecture.

This repository builds on concepts learned in the previous repository:

```text
helm-gitops-lab
```

which introduced:

* Helm
* ArgoCD Applications
* ApplicationSets
* Git Directory Generators
* Multi-Environment Deployments

This repository introduces:

* App of Apps Pattern
* Root Applications
* Cluster Bootstrap Architecture
* GitOps Hierarchies
* Platform-Level GitOps Management

---

# Architecture Overview

Traditional Application Deployment:

```text
Git
  ↓
Application
  ↓
Workload
```

ApplicationSet Deployment:

```text
Git
  ↓
ApplicationSet
  ↓
Applications
  ↓
Workloads
```

App of Apps Deployment:

```text
Git Repository
      ↓
Root Application
      ↓
ApplicationSet
      ↓
Applications
      ↓
Helm Charts
      ↓
Kubernetes Resources
```

This allows an entire cluster to be reconstructed from Git.

---

# Repository Structure

```text
cluster-bootstrap/

└── applications
    └── nginx-appset.yaml
```

---

# What Is Stored In This Repository?

This repository stores:

* ArgoCD Applications
* ArgoCD ApplicationSets
* Platform Bootstrap Configuration

This repository does NOT store:

* Helm Charts
* Application Source Code
* Environment Values

Those remain in:

```text
helm-gitops-lab
```

---

# Root Application

The Root Application is the only object manually applied to the cluster.

Purpose:

* Watches this repository
* Synchronizes Application definitions
* Creates ApplicationSets
* Maintains desired state

Architecture:

```text
root-app
    ↓
nginx-appset
    ↓
nginx-dev
nginx-qa
nginx-prod
nginx-perf
```

---

# Root Application YAML

Create:

```bash
nano root-app.yaml
```

Contents:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: root-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/<username>/cluster-bootstrap.git
    targetRevision: HEAD
    path: applications

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Replace:

```text
<username>
```

with your GitHub username.

---

# ApplicationSet

Current repository contains:

```text
nginx-appset.yaml
```

The ApplicationSet uses:

* Git Directory Generator
* Go Templates
* Helm Integration

Purpose:

```text
Git Directories
      ↓
Generate Applications
      ↓
Deploy Workloads
```

---

# ApplicationSet YAML

Create:

```bash
nano applications/nginx-appset.yaml
```

Contents:

```yaml
Copy from this repo
```

---

# Environment Discovery

ApplicationSet scans:

```text
environment/*
```

inside:

```text
helm-gitops-lab
```

Example:

```text
environment/
├── dev/
├── qa/
├── prod/
└── perf/
```

Generated Applications:

```text
nginx-dev
nginx-qa
nginx-prod
nginx-perf
```

---

# Bootstrap Procedure

## Prerequisites

* Kubernetes Cluster
* ArgoCD Installed
* GitHub Repository Accessible
* helm-gitops-lab Repository Available

---

## Step 1 - Verify ArgoCD

```bash
kubectl get pods -n argocd
```

Expected:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-applicationset-controller
```

---

## Step 2 - Create Namespaces

```bash
kubectl create namespace dev
kubectl create namespace qa
kubectl create namespace prod
kubectl create namespace perf
```

Verify:

```bash
kubectl get ns
```

---

## Step 3 - Deploy Root Application

Apply:

```bash
kubectl apply -f root-app.yaml
```

Verify:

```bash
kubectl get applications -n argocd
```

Expected:

```text
root-app
```

---

## Step 4 - Verify ApplicationSet Creation

Wait for reconciliation.

Verify:

```bash
kubectl get applicationsets -n argocd
```

Expected:

```text
nginx-appset
```

---

## Step 5 - Verify Generated Applications

```bash
kubectl get applications -n argocd
```

Expected:

```text
root-app
nginx-dev
nginx-qa
nginx-prod
nginx-perf
```

---

## Step 6 - Verify Deployments

```bash
kubectl get deploy -A
```

Expected:

```text
deployments running in:

dev
qa
prod
perf
```

---

# Auto Sync Demonstration

Modify:

```text
environment/dev/values.yaml
```

Example:

```yaml
replicaCount: 4
```

Commit and push.

Verify:

```bash
kubectl get deploy -n dev
```

ArgoCD automatically updates the deployment.

---

# Self Heal Demonstration

Manually scale:

```bash
kubectl scale deployment nginx-dev-nginx-helm \
--replicas=10 \
-n dev
```

Verify:

```bash
kubectl get deploy -n dev
```

After reconciliation:

```text
replicas restored from Git
```

---

# Prune Demonstration

Delete deployment from Git.

Commit and push.

Verify:

```bash
kubectl get deploy -A
```

ArgoCD automatically removes deleted resources.

---

# Automatic Environment Discovery

Create:

```bash
mkdir -p environment/staging
```

Create:

```bash
nano environment/staging/values.yaml
```

Example:

```yaml
replicaCount: 4
```

Commit and push.

Create namespace:

```bash
kubectl create namespace staging
```

Verify:

```bash
kubectl get applications -n argocd
```

Expected:

```text
nginx-staging
```

No ApplicationSet modifications required.

---

# Common Issue Encountered

ArgoCD 3.x requires:

```yaml
goTemplate: true
```

Incorrect:

```yaml
{{path.basename}}
```

Correct:

```yaml
{{ .path.basename }}
```

Incorrect:

```yaml
{{path.path}}
```

Correct:

```yaml
{{ .path.path }}
```

Without this configuration:

```text
{{path.path}}/values.yaml
```

remains unresolved and causes Helm rendering failures.

---

# Cleanup Procedure

Delete Root Application:

```bash
kubectl delete application root-app -n argocd
```

Verify:

```bash
kubectl get applications -n argocd
```

Expected:

```text
No resources found
```

Delete ApplicationSet if still present:

```bash
kubectl delete applicationset nginx-appset -n argocd
```

Delete namespaces:

```bash
kubectl delete namespace dev
kubectl delete namespace qa
kubectl delete namespace prod
kubectl delete namespace perf
kubectl delete namespace staging
```

Verify cleanup:

```bash
kubectl get applications -n argocd
kubectl get applicationsets -n argocd
```

Expected:

```text
No resources found
```

---

# Learning Outcomes

After completing this lab you should understand:

* App of Apps Pattern
* Root Applications
* ApplicationSets
* Git Directory Generators
* Go Templates
* Helm Integration
* Multi-Environment GitOps
* Cluster Bootstrapping
* GitOps Hierarchies
* Automated Kubernetes Management

---

# Next Steps

Recommended next topics:

* ArgoCD Projects
* Multi-Cluster GitOps
* CI/CD + GitOps Integration
* ArgoCD Image Updater
* Argo Rollouts
* Progressive Delivery
* Platform Engineering Workflows

This repository represents a production-style cluster bootstrap architecture and serves as the foundation for larger GitOps-managed Kubernetes environments.
