# homelab-kube

My personal homelab with k3s, managed by Flux CD.

## Table of Contents
- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Key Components](#key-components)
  - [GitOps with Flux CD](#gitops-with-flux-cd)
  - [Networking with MetalLB](#networking-with-metallb)
  - [Certificate Management (cert-manager)](#certificate-management-cert-manager)
  - [Applications](#applications)
    - [Glance](#glance)
  - [Monitoring Stack](#monitoring-stack)
- [Coming Soon](#coming-soon)
  - [n8n](#n8n)
  - [Backup](#backup)
  - [Advanced Networking](#advanced-networking)
- [Security Notes](#security-notes)
- [Contact](#contact)
- [References](#references)

## Overview
This repository contains everything needed to run a homelab Kubernetes cluster with k3s, leveraging GitOps via Flux CD. It includes load balancing, TLS, monitoring, and a dashboard application.

## Repository Structure

```
.
├── apps/
│   └── base/
│       ├── glance/                  # Dashboard application
│       ├── certmanager/             # TLS management
├── clusters/
│   └── staging/
│       └── apps.yaml                # Flux kustomization for staging
├── infrastructure/
│   └── base/
│       └── metallb/                 # MetalLB load balancer configs
├── monitoring/                      # Monitoring stack (Prometheus, Grafana)
└── namespaces/                      # Namespace definitions
```
## Technologies Used
This project utilizes a wide range of technologies:

Virtualization: Proxmox VE
Operating System: Ubuntu Server from cloud-init template
Configuration Management: Ansible
Containerization: Docker, Docker Compose
Orchestration: k3s (a lightweight Kubernetes distribution)
GitOps: FluxCD
Ingress & Networking: Nginx, MetalLB
Service Mesh: Tailscale
Monitoring & Alerting: Prometheus, Grafana
Log Management: (Not currently implemented)
Security: Cert-Manager (for TLS certificates)
Secrets Backend: -
DNS: Pihole
Applications: Home Assistant, Nginx Proxy Manager, Portainer, Plex, Servarr stack, OwnCloud, and many more.

## Getting Started

1. **Install k3s** on your hardware.
2. **Install Flux CD** and bootstrap it with your repo.
3. **Apply base namespaces** from `/namespaces`.
4. **Configure MetalLB** with suitable IP range for your network.
5. **Configure cert-manager** for TLS (update emails and secrets as needed).
6. **Deploy Glance and other apps** by syncing Flux or applying manifests.
7. **Monitor and manage** via Grafana and the Glance dashboard.

## Key Components

### GitOps with Flux CD
- All deployments and updates are managed by Flux CD.
- Regular sync intervals and pruning to keep cluster state up-to-date.

### Networking with MetalLB
- **Purpose:** Provides LoadBalancer services for bare metal clusters.
- **Configuration:**
  - MetalLB installed via Helm with resource limits.
    - [HelmRelease Example](https://github.com/eurus9696/homelab-kube/blob/main/infrastructure/base/metallb/helmrelease.yaml)
  - IP Address Pool: `192.168.1.240-192.168.1.245`
    - [IPAddressPool Example](https://github.com/eurus9696/homelab-kube/blob/main/infrastructure/base/metallb/ipaddresspool.yaml)
  - L2 Advertisement for IP pool.
  - Helm repo: [metallb.github.io/metallb](https://github.com/eurus9696/homelab-kube/blob/main/infrastructure/base/metallb/helmrepository.yaml)

### Certificate Management (cert-manager)
- **Purpose:** Manages TLS certificates for your cluster and applications.
- **Features:**
  - Let's Encrypt staging issuer.
  - Internal CA for services.
  - Auto-renewal and secret management.
  - Integrated with Ingress for secure access.
  - Example resources: [cert-manager HelmRelease](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/certmanager/helmrelease.yaml), [ClusterIssuer](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/certmanager/cluster-issuer-staging.yaml)

### Applications

#### Glance
A customizable dashboard for your homelab.

**Features:**
- Deployed as a single replica using the official image.
- Configurable via a large ConfigMap ([config-map.yaml](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/glance/config-map.yaml)):
  - Widgets for calendar, RSS feeds, to-do list, hacker-news, lobsters, YouTube channels, Reddit subreddits, weather, GitHub releases, bookmarks.
- Secured by TLS ([certificate-glance.yaml](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/certmanager/tls/certificate-glance.yaml)):
  - Hosted on `glance.home.lab` (internal DNS with piHole).
- Ingress rules for HTTPS ([ingress.yaml](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/glance/ingress.yaml)).
- Service as ClusterIP ([service.yaml](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/glance/service.yaml)).
- Deployment with resource limits ([deployment.yaml](https://github.com/eurus9696/homelab-kube/blob/main/apps/base/glance/deployment.yaml)):
  - Requests: 90m CPU, 200Mi memory
  - Limits: 100m CPU, 300Mi memory
  - Timezone and secret token injected via environment.

### Monitoring Stack
- **Kube Prom Stack**:
  - Deployed via Helm.
  - Persistent volume (5Gi) for dashboards and data.
  - Admin credentials configurable (replace default).
  - TLS enabled via cert-manager.
  - Resource limits enforced.
  - Service type: ClusterIP.
  - Sidecar for dashboards and datasources.
- **Prometheus**: (details not shown in search, but implied as part of common kube homelabs)

## Coming Soon

### n8n
Workflow automation with n8n will be integrated into the homelab stack, enabling you to automate tasks and connect various services.

## Security Notes
- Replace all default admin credentials with secure values.
- Use sealed secrets for sensitive data (e.g. tokens).
- Update email addresses for certificate management.

## Contact
For questions or issues, please contact: midhun25302@gmail.com
or just send me a PR

## References & Source
- [View the full codebase on GitHub](https://github.com/eurus9696/homelab-kube)
- [Explore MetalLB resources](https://github.com/metallb/metallb)
- [Explore Glance resources](https://github.com/glanceapp/glance)

---
