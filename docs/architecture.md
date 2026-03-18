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
            NP[Network Policies]
            CM[cert-manager]
        end

        subgraph Infrastructure
            EDNS[ExternalDNS]
            LPP[local-path-provisioner]
            RL[Reloader]
        end

        subgraph Monitoring
            PROM[Prometheus/Grafana]
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
    NP -->|restrict| Cluster
    VEL -->|backup| EXT
```

## Layers

**External** — Cloudflare provides DNS, CDN, and DDoS protection. A `cloudflared` tunnel connects the cluster to the internet without exposing any ports.

**Ingress** — ingress-nginx handles HTTP routing. MetalLB assigns LoadBalancer IPs on the local network. ExternalDNS synchronizes Ingress hostnames to Cloudflare DNS records.

**Security** — Network policies default to deny-all with explicit per-service allow rules enforced by Flannel's built-in NetworkPolicy controller. cert-manager provisions TLS certificates via Let's Encrypt. All secrets in Git are encrypted with SOPS + age.

**Infrastructure** — local-path-provisioner (k3s built-in) provides node-local persistent volumes. Reloader watches ConfigMaps and Secrets to trigger rolling restarts.

**Monitoring** — Prometheus scrapes metrics, Grafana visualizes them, Alertmanager routes alerts.

**Apps** — AdGuard Home for DNS-level ad blocking, Home Assistant for home automation, with room to grow.

## Network Flow

DNS resolution works in two paths. For remote access, Cloudflare resolves the public domain to its edge, which routes through the `cloudflared` tunnel to ingress-nginx inside the cluster. For local access, AdGuard Home or a local DNS server resolves `*.nlab.casa` to the MetalLB LoadBalancer IP, hitting ingress-nginx directly. In both cases, ingress-nginx terminates TLS (certs from cert-manager) and routes to the appropriate backend service.

## Security Model

Every namespace starts with a default-deny NetworkPolicy. Services must explicitly declare what ingress and egress they need. All secrets in Git are encrypted with SOPS using age keys. TLS is enforced on all ingress via cert-manager with Let's Encrypt.

## Backup Strategy

Velero runs on a daily schedule using file-system backup (kopia) to capture PVC data from local-path-provisioner volumes. Recovery: install Velero on fresh cluster, point at backup target, run `velero restore create --from-backup <latest>`.
