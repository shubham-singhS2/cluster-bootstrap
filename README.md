# Cluster Bootstrap Repository

## Overview

This repository implements a GitOps-based Kubernetes bootstrap architecture using ArgoCD.

The design follows the App of Apps pattern, where a single ArgoCD Root Application manages all platform applications through Git.

Architecture:

Git Repository
→ Root Application
→ ApplicationSet
→ Generated Applications
→ Helm Charts
→ Kubernetes Resources

The objective is to bootstrap and manage an entire Kubernetes cluster from Git with minimal manual intervention.

---

## Repository Structure

```text
cluster-bootstrap/
└── applications/
    └── nginx-appset.yaml
```

### applications/

Contains ArgoCD Application and ApplicationSet manifests that define what should exist in the cluster.

---

## Components

### Root Application

The Root Application is manually created once.

Purpose:

* Monitors this repository
* Synchronizes manifests under `applications/`
* Creates and manages child resources

Example:

```text
root-app
    ↓
nginx-appset
```

---

### ApplicationSet

The repository currently contains:

```text
nginx-appset
```

The ApplicationSet uses:

* Git Generator
* Go Templates
* Helm

Purpose:

* Automatically discover environments from Git
* Generate ArgoCD Applications
* Reduce repetitive application definitions

---

## Environment Discovery

The ApplicationSet scans:

```text
environment/*
```

inside the application repository.

Example:

```text
environment/
├── dev/
├── qa/
├── prod/
└── perf/
```

Each directory represents one deployment environment.

When a new directory is added:

```text
environment/staging/
```

the ApplicationSet automatically generates:

```text
nginx-staging
```

without modifying the ApplicationSet itself.

---

## ArgoCD Features Used

### Auto Sync

Automatically applies Git changes to the cluster.

### Self Heal

Restores resources when manual modifications create configuration drift.

Example:

```bash
kubectl scale deployment ...
```

ArgoCD automatically restores the desired state.

### Prune

Removes resources that are deleted from Git.

---

## Go Template Usage

ArgoCD version:

```text
v3.x
```

ApplicationSet uses:

```yaml
goTemplate: true
```

Template examples:

```yaml
{{ .path.basename }}
{{ .path.path }}
```

---

## Bootstrap Procedure

### Prerequisites

* Kubernetes Cluster
* ArgoCD Installed
* Git Repository Accessible

### Step 1

Deploy Root Application:

```bash
kubectl apply -f root-app.yaml
```

### Step 2

Verify Root Application:

```bash
kubectl get applications -n argocd
```

### Step 3

Verify ApplicationSet:

```bash
kubectl get applicationsets -n argocd
```

### Step 4

Verify Generated Applications:

```bash
kubectl get applications -n argocd
```

---

## Tested Scenarios

Verified:

* Root Application synchronization
* ApplicationSet generation
* Git Directory Generator
* Helm deployment
* Multi-environment deployment
* Auto Sync
* Self Heal
* Prune
* Environment auto-discovery

Example:

Adding:

```text
environment/perf/
```

automatically generated:

```text
nginx-perf
```

without any ApplicationSet modifications.

---

## Learning Outcomes

This repository demonstrates:

* App of Apps pattern
* ApplicationSets
* Git Directory Generator
* Helm integration
* Multi-environment GitOps
* Automated Kubernetes deployment using ArgoCD

---

## Future Enhancements

Potential future additions:

* ArgoCD Projects
* Multi-cluster deployments
* CI/CD integration
* ArgoCD Image Updater
* Argo Rollouts
* Progressive Delivery
* Cluster add-on management
* Platform bootstrap automation

```
```
