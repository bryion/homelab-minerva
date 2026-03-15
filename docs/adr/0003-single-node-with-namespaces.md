# ADR-0003: Single Node with Namespace-Based Separation

## Status

Accepted

## Date

2026-03-15

## Context

We want some degree of environment separation (e.g., staging vs production workloads) on a single physical node (i3-10100, 64GB RAM). Full VM-based isolation would consume resources on hypervisor overhead. Running multiple clusters adds operational complexity.

Options considered:

- **Proxmox VMs** — hard isolation, but hypervisor overhead wastes RAM and CPU on a resource-constrained single node.
- **Multiple k3s clusters** — strong separation, but doubles operational burden (two control planes, two sets of monitoring, two Flux instances).
- **Kubernetes namespace separation** — logical isolation via namespaces, enforced by RBAC, ResourceQuotas, and NetworkPolicies.

## Decision

Use Kubernetes namespace separation. Flux Kustomizations target separate namespaces for different environments. ResourceQuotas enforce memory and CPU boundaries per namespace. NetworkPolicies prevent cross-namespace traffic unless explicitly allowed.

## Consequences

- No hard isolation between environments — a kernel exploit could cross namespace boundaries. This is acceptable for a homelab.
- Simple to manage — one cluster, one control plane, one monitoring stack.
- All 64GB RAM and 4 CPU cores are available to workloads without hypervisor tax.
- ResourceQuotas prevent a runaway staging workload from starving production.
