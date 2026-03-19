# Repo Refactor: Align with Single-Node Essentials

**Date:** 2026-03-18
**Status:** Draft

## Problem

The repo was designed for a comprehensive Kubernetes homelab with 8 phases and many infrastructure components (MetalLB, ExternalDNS, Cloudflare tunnels, Velero, NetworkPolicies, Kyverno, etc.). The actual goal is simpler: run services like Mealie, Jellyfin, and AdGuard on a single node, accessible at `service.nlab.casa` with valid TLS, internal-only. The repo needs to be stripped down to match.

## Goals

- Deploy a single-node k3s cluster on bare metal (Intel i3-10100, 64GB RAM, 4TB NVMe)
- Run services accessible at `service-name.nlab.casa` with valid Let's Encrypt TLS (no browser warnings)
- Internal access only; remote access via Tailscale
- Use k3s for learning purposes
- Keep the stack minimal; add complexity only when needed

## Constraints

- `wiki.nlab.casa` is a Cloudflare Pages deployment and must not be affected
- DNS is managed via Cloudflare; per-service A records (not wildcard) to avoid colliding with `wiki.nlab.casa`
- Single-node: no HA, no multi-master, no distributed storage

## Architecture

### Layers (bottom to top)

1. **OS:** Ubuntu 24.04 LTS, hardened via Ansible (SSH key-only, UFW, unattended-upgrades, sysctl tuning)
2. **Cluster:** k3s single node, Flannel CNI, local-path-provisioner (built-in)
3. **GitOps:** Flux CD reconciles cluster state from this repo
4. **Ingress + TLS:** ingress-nginx routes by hostname on ports 80/443 (hostPort). cert-manager provisions Let's Encrypt certs via Cloudflare DNS-01 challenge.
5. **Ops:** Reloader (auto-restart on config/secret changes), kube-prometheus-stack (Prometheus, Grafana, Alertmanager)
6. **Apps:** AdGuard Home (DNS ad blocking), Tailscale (remote access), future apps (Mealie, Jellyfin)

### Firewall (UFW) Implications

The Ansible bootstrap playbook configures UFW with a default-deny policy. The following ports must be opened beyond what the playbook currently allows:

- **80/tcp, 443/tcp** — ingress-nginx hostPort (Phase 1)
- **53/tcp, 53/udp** — AdGuard Home DNS (Phase 3)

### DNS Flow

1. Per-service Cloudflare A records (managed via Terraform) point `service.nlab.casa` to node's LAN IP
2. Browser resolves `service.nlab.casa` to LAN IP, hits ingress-nginx on the node
3. ingress-nginx routes by hostname to the correct Kubernetes service
4. cert-manager has provisioned a valid Let's Encrypt cert via DNS-01 (no browser warnings)
5. Traffic never leaves the local network; Cloudflare only serves DNS responses

### What's Not Included (Deferred)

- MetalLB (ingress-nginx uses hostPort on single node)
- ExternalDNS (Terraform manages DNS records)
- Cloudflare tunnels (Tailscale replaces this)
- Velero / backups
- NetworkPolicies / security hardening
- Flux notifications
- Kyverno / policy enforcement

## Repo Structure

```
homelab-minerva/
├── .github/                          # CI (lint, validate)
├── .pre-commit-config.yaml
├── .sops.yaml
├── commitlint.config.js
├── Taskfile.yml                      # Trim to relevant tasks
├── docs/
│   ├── adr/                          # 0002, 0003, 0006 (remove superseded/deferred)
│   ├── architecture.md               # Rewrite to match this design
│   └── status.md                     # Rewrite with new phase plan
├── ansible/
│   ├── inventory/hosts.yml
│   └── playbooks/
│       ├── ubuntu-bootstrap.yml
│       └── k3s-upgrade.yml
├── kubernetes/
│   ├── flux-system/                  # Managed by Flux
│   ├── infrastructure/
│   │   ├── controllers/
│   │   │   ├── ingress-nginx/
│   │   │   ├── cert-manager/
│   │   │   └── reloader/
│   │   └── configs/
│   │       └── cert-manager/         # Cloudflare DNS-01 ClusterIssuer
│   ├── monitoring/
│   │   └── kube-prometheus-stack/
│   └── apps/
│       ├── adguard/
│       └── tailscale/
└── terraform/
    └── cloudflare/                   # Per-service A records
```

### Removed from Current Repo

- `kubernetes/infrastructure/controllers/metallb/`
- `kubernetes/infrastructure/controllers/external-dns/`
- `kubernetes/infrastructure/controllers/calico/`
- `kubernetes/infrastructure/configs/metallb/`
- `kubernetes/infrastructure/configs/external-dns/`
- `kubernetes/infrastructure/configs/network-policies/`
- `kubernetes/infrastructure/configs/flux-notifications/`
- `kubernetes/infrastructure/backup/velero/`
- `kubernetes/apps/cloudflared/`
- `kubernetes/apps/home-assistant/`
- `docs/adr/0001-use-talos-linux.md` (superseded)
- `docs/adr/0005-kyverno-for-policy-enforcement.md` (superseded)

### Retained but Deferred

- `docs/adr/0004-network-policy-default-deny.md` — kept for future reference; NetworkPolicies are deferred, not abandoned. ADR-0006 references this decision.

## Phase Plan

### Phase 1: Platform

- Install Ubuntu 24.04 on bare metal, run Ansible bootstrap (OS hardening + k3s)
- Bootstrap Flux CD, verify GitOps reconciliation loop
- Deploy ingress-nginx (hostPort 80/443)
- Deploy cert-manager with Cloudflare DNS-01 ClusterIssuer
- Deploy Reloader
- Terraform: set up Cloudflare provider, create first A record for testing
- Verify: HTTPS works on a test ingress with valid Let's Encrypt cert

### Phase 2: Monitoring

- Deploy kube-prometheus-stack (Prometheus, Grafana, Alertmanager)
- Ingress for Grafana at `grafana.nlab.casa`
- Terraform: add `grafana.nlab.casa` A record
- Verify: Grafana accessible with valid TLS, cluster metrics flowing

### Phase 3: Apps + Access

- Deploy AdGuard Home at `adguard.nlab.casa`, expose DNS on port 53 via hostPort
- Terraform: add `adguard.nlab.casa` A record
- Point router DNS to node LAN IP for network-wide ad blocking
- Deploy Tailscale as subnet router, advertising cluster CIDRs
- Verify: ad blocking works network-wide, remote access works via Tailscale

### Phase 4 (Future)

- Mealie, Jellyfin, Home Assistant — each follows the same pattern: Helm release + ingress + Terraform A record

## Success Criteria

- Services accessible at `service.nlab.casa` with valid TLS, no browser warnings
- All cluster state defined in Git, reconciled by Flux
- Monitoring operational with Grafana dashboards for cluster health
- AdGuard blocking ads network-wide
- Remote access via Tailscale without exposing ports to the internet
- Repo contains only what's deployed or planned; no dead manifests
