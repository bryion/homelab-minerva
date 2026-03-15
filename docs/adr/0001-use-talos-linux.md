# ADR-0001: Use Talos Linux as the Node Operating System

## Status

Accepted

## Date

2026-03-15

## Context

We need an operating system for a single-node bare metal Kubernetes homelab running on an i3-10100 with 64GB RAM and 4TB NVMe storage. The cluster is managed remotely from an M4 MacBook, so API-driven management is essential. Key requirements: immutability for reproducibility, minimal attack surface, and efficient resource usage so the OS doesn't compete with workloads.

Options considered:

- **Ubuntu + k3s** — familiar and flexible, but mutable state leads to drift over time. SSH access is a security surface. Package manager and OS updates are a maintenance burden.
- **Proxmox + VMs** — strong isolation, but adds a hypervisor layer consuming resources. Over-engineered for a single-node setup running only Kubernetes.
- **Flatcar Container Linux** — immutable and container-focused, but still a general-purpose Linux with SSH. Requires Ignition configs and has a broader surface than needed.
- **Talos Linux** — purpose-built for Kubernetes, immutable, no SSH, entirely API-managed via `talosctl`. Minimal footprint.

## Decision

Use Talos Linux. It is immutable, has no SSH, is managed entirely via a gRPC API, and is purpose-built for running Kubernetes. Its minimal footprint (~80MB) leaves maximum resources for workloads.

## Consequences

- No SSH access for debugging — all troubleshooting goes through `talosctl` (logs, dmesg, dashboard).
- Cannot run non-Kubernetes workloads on the node (no Docker Compose, no bare-metal services).
- Ansible is only useful for initial bootstrap (writing the Talos image, generating configs). Ongoing management is via `talosctl` and Kubernetes APIs.
- OS upgrades are atomic and API-driven — no apt-get, no reboot-and-pray.
