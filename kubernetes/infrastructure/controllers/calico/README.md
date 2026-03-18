# Calico CNI

Calico is deployed here as the cluster CNI to enforce NetworkPolicies.

k3s is installed with `--flannel-backend=none --disable-network-policy`, which disables the default Flannel CNI and the built-in network policy controller. Calico replaces Flannel and provides a NetworkPolicy-aware data plane.

This is required by [ADR-0004](../../../docs/adr/0004-default-deny-network-policies.md), which mandates default-deny NetworkPolicies across all namespaces. Flannel does not implement the NetworkPolicy API — any policies created with Flannel as the CNI are silently ignored.

**Calico must be the first infrastructure component deployed** (before any workloads that depend on NetworkPolicies being enforced).
