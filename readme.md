# App Configuration – GitOps Deployment (ArgoCD + EKS)

This repository contains all Kubernetes manifests and GitOps configuration for deploying the application to Amazon EKS using ArgoCD.

It works alongside the `app-source-code` repository:
- `app-source-code` builds and pushes secure images to ECR
- This repository pulls those images and deploys them to the cluster

---

## Overview

This project implements a GitOps-based deployment strategy using ArgoCD to manage:

- Blue/green application deployments (v1 and v2)
- Kubernetes manifests (Deployments, Services, Ingress)
- Secrets management via AWS Secrets Manager
- Observability stack (Prometheus, Alertmanager, Grafana)

ArgoCD continuously syncs the desired state defined in this repository with the actual state in the EKS cluster.

---

## Key Features

### GitOps with ArgoCD

- Declarative Kubernetes manifests stored in Git
- Automatic synchronization to EKS
- Drift detection and self-healing
- Version-controlled deployments

When new images are pushed to ECR, ArgoCD pulls and applies updates to the cluster.

---

## Blue-Green Deployment Strategy

This repository enables blue-green deployments by running two versions of the application simultaneously:

- `v1` (current/live version)
- `v2` (new version)

### How it works

- Separate deployments for each version
- Services and ingress control traffic routing
- Both versions can run in parallel for validation
- Traffic can be shifted between versions without downtime

This allows:

- Safe rollouts
- Easy rollback
- Real-time comparison between versions

---

## Kubernetes Workloads

### Backend

- Multiple deployments (`v1`, `v2`)
- FastAPI-based application
- Health checks:
  - Readiness probe (`/health`)
  - Liveness probe (`/health`)
- Environment configuration via:
  - ConfigMaps (non-sensitive data)
  - Secrets (sensitive data)

### Frontend

- Separate deployments for `v1` and `v2`
- Exposed via Kubernetes services and ingress

---

## Ingress & Traffic Management

- AWS Load Balancer Controller provisions ALB
- Ingress resources route external traffic to services
- Supports controlled routing between backend/frontend versions

---

## Secrets Management (Secure by Design)

This project uses the **Secrets Store CSI Driver with AWS Secrets Manager**:

- No hardcoded secrets in Kubernetes manifests
- Secrets are dynamically mounted at runtime
- Synced into Kubernetes secrets when needed

### Flow

1. Secrets stored in AWS Secrets Manager
2. SecretProviderClass defines how to fetch them
3. CSI driver mounts secrets into pods
4. Kubernetes secret is created for app consumption

This ensures:

- Centralized secret management
- Rotation without redeploying code
- Reduced risk of secret leakage

---

## Configuration Management

- ConfigMaps store environment-specific values
- Separate configs for `v1` and `v2`
- Used to inject:
  - Deployment identifiers
  - Version metadata

---

## Observability & Monitoring

Integrated monitoring stack includes:

### Prometheus

- Metrics collection from services
- ServiceMonitor resources for scraping application metrics

### Alertmanager

- Alert routing and notification handling
- Custom alert rules defined for application health

### Grafana

- Dashboards for visualization
- Exposed via ingress

This provides:

- Real-time visibility into application performance
- Alerting on failures or anomalies
- Metrics for validating deployments

---

## Service Accounts & IAM Integration

- Kubernetes service accounts mapped to AWS IAM roles
- Enables secure access to AWS services (e.g., Secrets Manager)
- Follows least privilege principles

---

## Deployment Flow

1. CI pipeline builds and pushes images to ECR (from `app-source-code`)
2. Image tags are updated (manually or via automation)
3. ArgoCD detects changes in this repository
4. ArgoCD syncs manifests to EKS
5. Kubernetes updates pods with new images
6. Both versions (v1 and v2) run simultaneously
7. Traffic is routed based on ingress/service configuration

---

## High Availability & Reliability

- Multi-replica deployments
- Readiness and liveness probes
- Multi-AZ EKS cluster (provisioned via Terraform)
- Zero-downtime deployments via blue-green strategy

---

## What This Repo Does Not Do

This repository does not:

- Build Docker images
- Run CI/CD pipelines
- Perform security scans

Those responsibilities belong to the `app-source-code` repository.

---

## Why This Project Matters

This repository demonstrates production-level Kubernetes and GitOps practices:

- ArgoCD-driven continuous delivery
- Secure secret management with AWS integration
- Blue-green deployment strategy on EKS
- Observability with Prometheus and Grafana
- Separation of concerns between CI and CD