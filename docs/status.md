# Project Status: homelab-minerva

**Date:** 2026-03-15
**Current Phase:** Phase 0 (Complete)

## Phase 0 Checklist

- [x] All CLI tools installed (kubectl 1.35.2, flux 2.8.2, sops 3.12.1, age 1.3.1, task 3.49.1, pre-commit 4.5.1, kubeconform 0.7.0, ansible 2.15.13, yamllint 1.37.1, gh 2.88.1)
- [x] Age keypair generated (`~/.config/sops/age/keys.txt` exists, `SOPS_AGE_KEY_FILE` set in `~/.zshrc`)
- [x] SOPS encrypt/decrypt roundtrip works
- [x] Pre-commit hooks installed and passing (trim whitespace, fix EOL, check yaml, detect secrets)
- [x] Commitlint configured and blocking bad messages (`commitlint.config.js` extends `@commitlint/config-conventional`, commit-msg hook installed)
- [x] Repo structure matches scaffold (all 23 expected files/directories present)
- [x] Git remote connected to GitHub (`bryion/homelab-minerva`)
- [x] CI lint workflow exists (`.github/workflows/lint.yml` runs on PRs to main)
- [x] Taskfile with all commands (13 tasks: node, k3s, flux, sops, validate, velero)

## Architecture Overview

Single-node Kubernetes cluster running Ubuntu 24.04 LTS with k3s on bare metal, managed via Flux CD GitOps. External access is provided through Cloudflare tunnels (cloudflared) without exposing ports. Security follows a defense-in-depth model with default-deny network policies, SOPS-encrypted secrets, and cert-manager TLS. Monitoring uses the Prometheus/Grafana stack, with Velero handling disaster recovery backups.

## Phase Plan

| Phase | Description | Status |
|-------|-------------|--------|
| 0 | Workstation setup | COMPLETE |
| 1 | Ubuntu 24.04 + k3s on bare metal | NOT STARTED |
| 2 | Flux bootstrap + GitOps loop | NOT STARTED |
| 3 | Networking stack (MetalLB, ingress, certs, DNS) | NOT STARTED |
| 4 | Storage + first app (AdGuard) | NOT STARTED |
| 5 | Monitoring (Prometheus, Grafana) | NOT STARTED |
| 6 | Security hardening (default-deny NetworkPolicies) | NOT STARTED |
| 7 | Reliability + operations (Velero, resource quotas, Reloader) | NOT STARTED |
| 8 | Apps + remote access (cloudflared, Home Assistant) | NOT STARTED |

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
- **Ingress:** ingress-nginx, MetalLB, ExternalDNS
- **Security:** default-deny NetworkPolicies, cert-manager
- **Storage:** local-path-provisioner (k3s built-in)
- **Monitoring:** Prometheus, Grafana, Alertmanager
- **Backup:** Velero
- **Apps:** AdGuard Home, Home Assistant
- **Remote Access:** cloudflared (Cloudflare Tunnel)
- **Utilities:** Reloader

## Next Steps

Phase 1 begins with Ubuntu 24.04 LTS + k3s installation on the bare metal machine:

1. Install Ubuntu 24.04 LTS Server on the bare metal node
2. Update `ansible/inventory/hosts.yml` with the node IP
3. Run `task node:bootstrap` to harden the OS and install k3s
4. Verify cluster health with `kubectl get nodes` and `kubectl get pods -A`
