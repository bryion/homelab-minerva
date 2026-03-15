# ADR-0004: Default-Deny Network Policies

## Status

Accepted

## Date

2026-03-15

## Context

All pods in a flat Kubernetes network can communicate with any other pod by default. If a container is compromised, an attacker can move laterally to reach databases, monitoring endpoints, or other sensitive services without restriction.

Options considered:

- **No network policies** — rely entirely on application-level authentication and authorization. Simple but offers no defense in depth.
- **Calico/Cilium with advanced policies** — powerful L7 policies and DNS-aware rules, but introduces CNI-specific CRDs that reduce portability and add complexity.
- **Kubernetes-native NetworkPolicy with default deny** — uses the standard NetworkPolicy API. Works with any CNI that supports it. No vendor lock-in.

## Decision

Deploy a default-deny NetworkPolicy in every namespace that blocks all ingress and egress. Each service then declares explicit NetworkPolicy resources for the traffic it needs. Use only the standard Kubernetes NetworkPolicy API for portability across CNI implementations.

## Consequences

- Every new service requires a NetworkPolicy before it can send or receive traffic. This adds YAML per service but makes traffic flows explicit and auditable.
- Lateral movement risk is eliminated — a compromised pod can only reach what its NetworkPolicy allows.
- Debugging connectivity issues requires checking NetworkPolicies first (`kubectl get netpol -n <namespace>`).
- DNS egress (port 53 to kube-dns) must be explicitly allowed in every namespace, or pods cannot resolve service names.
