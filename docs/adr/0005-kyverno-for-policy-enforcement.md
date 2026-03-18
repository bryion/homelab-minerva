# ADR-0005: Kyverno for Policy Enforcement

## Status

Superseded

> Superseded 2026-03-18. In a single-author homelab, Kyverno enforces policies that the sole
> operator already follows manually. Pre-commit YAML linting provides equivalent shift-left
> safety without admission webhook complexity. May be revisited if the cluster becomes
> multi-user.

## Date

2026-03-15

## Context

We need to enforce security and operational standards across all workloads: no root containers, required resource limits, image source restrictions, and required labels for observability. These rules should be enforced at admission time, not just audited after the fact.

Options considered:

- **OPA Gatekeeper** — powerful and flexible, but policies are written in Rego (a domain-specific language with a steep learning curve). Constraint templates add indirection.
- **Kyverno** — Kubernetes-native: policies are Custom Resources written in YAML. Lower learning curve, built-in policy library, generates PolicyReport CRDs for compliance dashboards.
- **Manual code review only** — catches issues before merge but doesn't prevent kubectl apply or Flux from deploying non-compliant resources.

## Decision

Use Kyverno. Policies are Kubernetes CRDs written in familiar YAML, making them accessible without learning Rego. The built-in policy library covers common security baselines. PolicyReport resources integrate with Grafana for compliance visibility.

## Consequences

- Adds a validating webhook to the admission chain — slight latency increase on pod creation (typically <100ms).
- Policies must be tested in `Audit` mode before switching to `Enforce` to avoid blocking legitimate workloads.
- The team only needs to know YAML to write and maintain policies, lowering the barrier to contribution.
- PolicyReport CRDs provide a standard interface for compliance dashboards without custom tooling.
