# ADR-0002: Use Flux CD Over ArgoCD for GitOps

## Status

Accepted

## Date

2026-03-15

## Context

We need a GitOps operator to continuously reconcile Kubernetes cluster state from a Git repository. The operator must handle Kustomize overlays, HelmRelease resources, and SOPS-encrypted secrets. It runs on a single-node cluster with 4 CPU cores, so resource efficiency matters.

Options considered:

- **Flux CD** — lightweight, runs as a set of controllers, native Kustomize and Helm support, built-in SOPS decryption, no UI.
- **ArgoCD** — feature-rich with a web UI, supports multiple sync strategies, but heavier resource footprint (API server, repo server, Redis, UI).
- **Raw kubectl apply from CI** — simple but not GitOps. No drift detection, no automatic reconciliation, no self-healing.

## Decision

Use Flux CD. Its lighter resource footprint is better suited for a 4-core single node. It has native SOPS integration for secret decryption, first-class Kustomize support, and does not require a UI server.

## Consequences

- No visual dashboard for sync status — use `flux get all` CLI and Grafana dashboards fed by Flux's Prometheus metrics.
- Steeper initial learning curve compared to ArgoCD's UI-driven workflow.
- Reconciliation debugging is CLI-only via `flux logs` and `kubectl describe kustomization`.
- Notification controller can push alerts to Discord/Slack/GitHub for visibility into sync failures.
