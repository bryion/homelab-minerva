# ADR-0006: Switch from Talos Linux to Ubuntu 24.04 LTS + k3s

## Status

Accepted

## Date

2026-03-18

## Context

ADR-0001 selected Talos Linux as the node OS. At the time of this decision, no OS has been installed (Phase 0 is complete — workstation setup only). The switch carries zero migration cost.

Talos's no-SSH model proved to be a practical obstacle during homelab operation. Bare-metal debugging, hardware validation, and one-off maintenance tasks all require interactive access to the node. The `talosctl` API covers many cases but not all — e.g., running `nvme-cli`, checking BIOS settings, or recovering from a misconfigured network interface. Every such case would require booting a live USB, which breaks the automation model anyway.

Ubuntu 24.04 LTS provides:

- Familiar tooling for debugging and maintenance
- Broad Ansible support with mature modules (`community.general`, `ansible.posix`)
- Long-term support through April 2029
- A straightforward path to OS hardening (UFW, unattended-upgrades, SSH key-only auth)

k3s is the de facto single-node homelab Kubernetes distribution:

- Single binary install via official install script
- Minimal resource overhead (~512MB RAM)
- Production-grade Kubernetes API compatibility
- Active upstream maintenance by Rancher/SUSE
- Supports the same Flux CD / Helm / CRD workflows as full Kubernetes

## Decision

Replace Talos Linux with Ubuntu 24.04 LTS. Replace the Talos cluster with k3s. Use Ansible to provision the OS, harden it, and install k3s. All layers above the OS (Flux CD, Helm releases, Kubernetes manifests, SOPS, Terraform) are unchanged.

**CNI note:** k3s ships with Flannel, which does not enforce NetworkPolicies. Because ADR-0004 mandates default-deny NetworkPolicies, k3s is installed with `--flannel-backend=none --disable-network-policy` so that a NetworkPolicy-aware CNI (Calico) can be deployed via Flux in the infrastructure layer.

## Consequences

- SSH access is available for debugging and maintenance.
- OS is mutable — drift is possible, mitigated by Ansible idempotency and periodic re-runs.
- OS upgrades are handled by `unattended-upgrades` for security patches and `task node:upgrade` for full upgrades.
- Ansible becomes the primary node management tool (not just bootstrap).
- Calico must be deployed as the first infrastructure component before any NetworkPolicy-dependent workloads start.
- `talosctl` is removed from the prerequisites list.
