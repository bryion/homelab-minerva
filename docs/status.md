# Project Status: homelab-minerva

**Date:** 2026-03-18
**Current Phase:** Phase 0 (Complete)

## Phase 0 Checklist

- [x] All CLI tools installed (kubectl, flux, sops, age, task, pre-commit, kubeconform, ansible, yamllint)
- [x] Age keypair generated and SOPS configured
- [x] Pre-commit hooks installed and passing
- [x] Commitlint configured with conventional commits
- [x] Git remote connected to GitHub
- [x] CI lint workflow active on PRs
- [x] Taskfile with automation tasks
- [x] Architecture decisions documented (ADRs)
- [x] Repo refactored to essential components only

## Architecture Overview

Single-node k3s cluster on bare metal (Ubuntu 24.04 LTS), managed via Flux CD GitOps. Services accessible at `service.nlab.casa` with valid Let's Encrypt TLS (internal only). Remote access via Tailscale. DNS managed via Cloudflare A records (per-service, managed by Terraform).

## Phase Plan

| Phase | Description | Status |
|-------|-------------|--------|
| 0 | Workstation setup + repo scaffold | COMPLETE |
| 1 | Ubuntu 24.04 + k3s + Flux + ingress-nginx + cert-manager + Reloader | NOT STARTED |
| 2 | Monitoring (Prometheus, Grafana, Alertmanager) | NOT STARTED |
| 3 | Apps + access (AdGuard Home, Tailscale) | NOT STARTED |
| 4 | Future apps (Mealie, Jellyfin, etc.) | NOT STARTED |

## Hardware

| Component | Spec |
|-----------|------|
| CPU | Intel Core i3-10100 (4c/8t) |
| RAM | 64 GB DDR4-3200 |
| Storage | 4 TB NVMe (Crucial P3) |
| Case | Fractal Design Ridge Mini ITX |
| PSU | Silverstone SX500-G 500W SFX |
| Motherboard | MSI MPG B560I Gaming Edge WiFi |

## Tech Stack

### Deployed
- **Workstation:** kubectl, flux, sops, age, task, pre-commit, kubeconform, ansible, yamllint
- **CI/CD:** GitHub Actions (lint workflow), Renovate (dependency updates)
- **Secrets:** SOPS + age encryption

### Planned
- **OS:** Ubuntu 24.04 LTS + k3s
- **GitOps:** Flux CD
- **Ingress:** ingress-nginx (hostPort 80/443)
- **TLS:** cert-manager (Let's Encrypt via Cloudflare DNS-01)
- **DNS:** Cloudflare A records (managed via Terraform)
- **Storage:** local-path-provisioner (k3s built-in)
- **Monitoring:** Prometheus, Grafana, Alertmanager (kube-prometheus-stack)
- **Apps:** AdGuard Home
- **Remote Access:** Tailscale (subnet router)
- **Utilities:** Reloader

## Next Steps

Phase 1 begins with Ubuntu 24.04 LTS + k3s installation on the bare metal machine:

1. Install Ubuntu 24.04 LTS Server on the bare metal node
2. Update `ansible/inventory/hosts.yml` with the node IP
3. Run `task node:bootstrap` to harden the OS and install k3s
4. Bootstrap Flux CD with `task flux:bootstrap`
5. Deploy ingress-nginx, cert-manager, and Reloader
6. Set up Terraform for Cloudflare DNS A records
7. Verify HTTPS works with valid Let's Encrypt cert
