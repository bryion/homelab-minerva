# homelab-minerva

[![Validate](https://github.com/<user>/homelab-minerva/actions/workflows/lint.yml/badge.svg)](https://github.com/<user>/homelab-minerva/actions/workflows/lint.yml)
![Last Commit](https://img.shields.io/github/last-commit/<user>/homelab-minerva)
![License](https://img.shields.io/github/license/<user>/homelab-minerva)

> GitOps-managed single-node Kubernetes cluster on bare metal, powered by Ubuntu 24.04, k3s, and Flux CD.

## Architecture

```mermaid
graph LR
    subgraph External
        GH[GitHub Repo]
        CF[Cloudflare]
    end

    subgraph Cluster["Kubernetes (Ubuntu 24.04 / k3s)"]
        FLUX[Flux CD] -->|reconcile| INFRA[Infrastructure]
        FLUX -->|reconcile| APPS[Apps]

        subgraph Security
            NP[Network Policies]
        end

        subgraph Services
            NGINX[ingress-nginx]
            ADG[AdGuard]
            HA[Home Assistant]
        end
    end

    subgraph Backup
        VEL[Velero] -->|snapshots| EXT[External Target]
    end

    GH -->|sync| FLUX
    CF -->|tunnel| NGINX
    Security -.->|enforce| Cluster
```

## Hardware

| Component | Spec |
|-----------|------|
| CPU | Intel i3-10100 (4C/8T) |
| RAM | 64GB DDR4-3200 |
| Storage | 4TB NVMe |
| Case | Fractal Ridge ITX |
| PSU | Silverstone SX500-G |

## Tech Stack

| Layer | Tools |
|-------|-------|
| OS | Ubuntu 24.04 LTS + k3s |
| GitOps | Flux CD, Renovate |
| Networking | ingress-nginx, MetalLB, ExternalDNS, Cloudflare Tunnels |
| Security | SOPS/age, cert-manager, NetworkPolicies |
| Storage | local-path-provisioner (k3s built-in) |
| Backup | Velero |
| Monitoring | Prometheus, Grafana, Alertmanager |
| DNS | AdGuard Home |
| Automation | Home Assistant |
| Operations | Reloader, Flux notifications |
| IaC | Ansible (provisioning + maintenance), Terraform (Cloudflare) |

## Prerequisites

```
kubectl flux sops age task ansible pre-commit kubeconform velero
```

## Quick Start

1. **Generate an age key** — `task sops:age-keygen` and add the public key to `.sops.yaml`
2. **Configure SOPS** — set `SOPS_AGE_KEY_FILE` to point to your private key
3. **Bootstrap node** — update `ansible/inventory/hosts.yml` with your node IP and run `task node:bootstrap`
4. **Bootstrap Flux** — fill in `GITHUB_OWNER` in `Taskfile.yml` and run `task flux:bootstrap`
5. **Verify** — `flux get all` and `kubectl get nodes`

## Repo Structure

```
homelab-minerva/
├── .github/              # CI workflows, PR template, Renovate config
├── docs/                 # Architecture docs and ADRs
├── ansible/              # Inventory and provisioning playbooks
├── kubernetes/
│   ├── flux-system/      # Flux bootstrap (managed by Flux)
│   ├── infrastructure/
│   │   ├── controllers/  # cert-manager, ingress-nginx, external-dns, metallb, reloader
│   │   ├── configs/      # cert-manager issuers, metallb pools, network policies
│   │   ├── storage/      # local-path-provisioner config
│   │   └── backup/       # Velero
│   ├── monitoring/       # kube-prometheus-stack
│   └── apps/             # adguard, home-assistant, cloudflared
└── terraform/            # Cloudflare DNS and tunnel config
```

## Documentation

- [Architecture Overview](docs/architecture.md)
- [Architecture Decision Records](docs/adr/)

## License

[MIT](LICENSE)
