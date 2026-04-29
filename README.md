# aks-poc-001

## Overview

This repository contains Kubernetes manifests for deploying a **Node.js 24** application on **Azure Kubernetes Service (AKS)**. The manifests demonstrate three different container image sourcing strategies — from a private Azure Container Registry (ACR), from the authenticated Red Hat registry, and from the publicly accessible Red Hat Universal Base Image (UBI) registry. Each approach targets a different team setup or compliance requirement.

All deployments expose the application via a `LoadBalancer` Service on port 80 and run two replicas with CPU/memory resource limits, readiness probes, and liveness probes configured.

---

## YAML Files

### 1. `nodejs-deployment.yaml` — ACR-Hosted S2I Image

| Property | Value |
|---|---|
| **Namespace** | Creates `indra-poc` |
| **Image** | `oapoc001.azurecr.io/rhel8/nodejs-24:s2i` |
| **Image Pull Secret** | `acr-secret` |
| **Replicas** | 2 |
| **Service Type** | LoadBalancer |

**Description:**  
This manifest creates the `indra-poc` namespace and deploys the application using a custom image stored in a private **Azure Container Registry (ACR)**. The image was built using the **Source-to-Image (S2I)** toolchain and pushed to the team's ACR instance (`oapoc001.azurecr.io`). Authentication to ACR is handled via the `acr-secret` Kubernetes secret.

**When to use:**
- Your team builds custom images and stores them in a private ACR instance.
- You need full control over the base image (e.g., pre-installed dependencies, custom configuration, or security hardening applied during the S2I build).
- You want to avoid dependency on external registries at runtime.
- This is the **recommended starting point** for new clusters — it also creates the namespace.

---

### 2. `nodejs-deployment-v2.yaml` — Official Red Hat Registry Image

| Property | Value |
|---|---|
| **Namespace** | Uses existing `indra-poc` (not created) |
| **Image** | `registry.redhat.io/rhel8/nodejs-24` |
| **Image Pull Secret** | `redhat-pull-secret` |
| **Replicas** | 2 |
| **Service Type** | LoadBalancer |

**Description:**  
This manifest deploys the application using the official **Red Hat Node.js 24** image pulled directly from `registry.redhat.io`. This registry requires a valid **Red Hat subscription** and the credentials must be stored in the `redhat-pull-secret` Kubernetes secret. The namespace `indra-poc` must already exist before applying this manifest (e.g., created by `nodejs-deployment.yaml`).

**When to use:**
- Your organization has an active **Red Hat subscription** and wants to use officially supported, subscription-gated images.
- Compliance or security policies require using images from the authenticated Red Hat registry rather than a public endpoint.
- You want to benefit from Red Hat's full support lifecycle and errata updates for the base image.
- Use this in **regulated or enterprise environments** where image provenance and support contracts matter.

---

### 3. `nodejs-deployment-v3.yaml` — Red Hat UBI Public Registry Image

| Property | Value |
|---|---|
| **Namespace** | Uses existing `indra-poc` (not created) |
| **Image** | `registry.access.redhat.com/ubi8/nodejs-24` |
| **Image Pull Secret** | `acr-secret` |
| **Replicas** | 2 |
| **Service Type** | LoadBalancer |

**Description:**  
This manifest deploys the application using the **Red Hat Universal Base Image (UBI)** pulled from `registry.access.redhat.com`, which is **publicly accessible without authentication**. UBI images are freely redistributable and designed for container-first workloads. The `acr-secret` is listed as an image pull secret in this manifest (inherited from the cluster configuration), but it is not required for pulling this specific public image. The namespace `indra-poc` must already exist.

**When to use:**
- You want a **freely available, no-subscription** Red Hat-based base image.
- You are building or testing in environments without a Red Hat subscription.
- You prefer a **minimal, production-grade** base image that is publicly redistributable (e.g., for open-source projects or shared demo environments).
- Use this for **dev/test, CI pipelines, or open-source scenarios** where a subscription is not available or desired.

---

## Comparison Summary

| Feature | `nodejs-deployment.yaml` | `nodejs-deployment-v2.yaml` | `nodejs-deployment-v3.yaml` |
|---|---|---|---|
| **Image Registry** | Private ACR (S2I) | `registry.redhat.io` | `registry.access.redhat.com` |
| **Authentication Required** | ACR secret | Red Hat subscription | None (public) |
| **Creates Namespace** | ✅ Yes | ❌ No | ❌ No |
| **Pull Secret** | `acr-secret` | `redhat-pull-secret` | `acr-secret` |
| **Best For** | Custom/hardened images | Enterprise + RH subscription | Dev/test, OSS, no subscription |
| **Apply Order** | First | After namespace exists | After namespace exists |

---

## Prerequisites

- An AKS cluster up and running.
- `kubectl` configured to point at your cluster.
- Required Kubernetes secrets created in the `indra-poc` namespace:
  - `acr-secret` — for pulling from your private ACR (used by v1 and v3).
  - `redhat-pull-secret` — for pulling from `registry.redhat.io` (used by v2).

## Usage

```bash
# Deploy v1 (also creates the namespace)
kubectl apply -f nodejs-deployment.yaml

# Deploy v2 (namespace must already exist)
kubectl apply -f nodejs-deployment-v2.yaml

# Deploy v3 (namespace must already exist)
kubectl apply -f nodejs-deployment-v3.yaml
```