# Architecture

## Overview

```mermaid
graph TB
    subgraph External
        CF[Cloudflare DNS/CDN]
        Git[GitHub Repository]
    end

    subgraph Cluster["Kubernetes Cluster (Ubuntu 24.04 / k3s)"]
        subgraph Ingress
            CFD[cloudflared]
            NGINX[ingress-nginx]
            MLB[MetalLB]
        end

        subgraph Security
            KYV[Kyverno]
            NP[Network Policies]
            CM[cert-manager]
            TRIVY[Trivy Operator]
        end

        subgraph Infrastructure
            EDNS[ExternalDNS]
            LH[Longhorn]
            RL[Reloader]
        end

        subgraph Monitoring
            PROM[Prometheus/Grafana]
            LOKI[Loki]
            AM[Alertmanager]
        end

        subgraph Apps
            ADG[AdGuard Home]
            HA[Home Assistant]
        end

        subgraph GitOps
            FLUX[Flux CD]
            VEL[Velero]
        end
    end

    subgraph Backup
        EXT[External Backup Target]
    end

    CF -->|tunnel| CFD
    CFD --> NGINX
    NGINX --> Apps
    MLB -->|LoadBalancer IPs| NGINX
    EDNS -->|DNS records| CF
    CM -->|TLS certs| NGINX
    Git -->|sync| FLUX
    FLUX -->|reconcile| Cluster
    PROM -->|scrape| Cluster
    KYV -->|enforce| Cluster
    NP -->|restrict| Cluster
    TRIVY -->|scan| Cluster
    VEL -->|backup| EXT
```

## Layers

**External** — Cloudflare provides DNS, CDN, and DDoS protection. A `cloudflared` tunnel connects the cluster to the internet without exposing any ports.

**Ingress** — ingress-nginx handles HTTP routing. MetalLB assigns LoadBalancer IPs on the local network. ExternalDNS synchronizes Ingress hostnames to Cloudflare DNS records.

**Security** — Kyverno enforces admission policies (no root, required labels, image allowlists). Network policies default to deny-all with explicit per-service allow rules. cert-manager provisions TLS certificates via Let's Encrypt. Trivy Operator continuously scans container images for vulnerabilities.

**Infrastructure** — Longhorn provides replicated persistent storage. Reloader watches ConfigMaps and Secrets to trigger rolling restarts when configuration changes.

**Monitoring** — Prometheus scrapes metrics, Grafana visualizes them, Loki aggregates logs, Alertmanager routes alerts.

**Apps** — AdGuard Home for DNS-level ad blocking, Home Assistant for home automation, with room to grow.

## Network Flow

DNS resolution works in two paths. For remote access, Cloudflare resolves the public domain to its edge, which routes through the `cloudflared` tunnel to ingress-nginx inside the cluster. For local access, AdGuard Home or a local DNS server resolves `*.nlab.casa` to the MetalLB LoadBalancer IP, hitting ingress-nginx directly. In both cases, ingress-nginx terminates TLS (certs from cert-manager) and routes to the appropriate backend service.

## Security Model

Every namespace starts with a default-deny NetworkPolicy. Services must explicitly declare what ingress and egress they need. Kyverno ClusterPolicies enforce: no privileged containers, no root users, required resource limits, image registries restricted to ghcr.io/docker.io/quay.io, and required `app.kubernetes.io` labels. All secrets in Git are encrypted with SOPS using age keys. TLS is enforced on all ingress via cert-manager with Let's Encrypt.

## Backup Strategy

Velero runs on a daily schedule backing up all namespaces and cluster-scoped resources. Persistent volume data is captured via Longhorn snapshots. Recovery procedure: install Velero on a fresh cluster, point it at the backup target, run `velero restore create --from-backup <latest>`, then verify workloads come up and PVCs rebind to restored Longhorn volumes.
