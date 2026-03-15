ClusterPolicy resources for Kyverno admission control.

Planned policies:

- **require-resource-limits** — all containers must specify CPU and memory requests and limits
- **disallow-privilege-escalation** — no privileged containers or containers running as root
- **require-labels** — all Deployments must have `app.kubernetes.io/name` and `app.kubernetes.io/component` labels
- **restrict-image-registries** — only allow images from `ghcr.io`, `docker.io`, and `quay.io`
