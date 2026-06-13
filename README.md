# Hetzner Platform

GitOps repository for the Hetzner K3s cluster.

This repository owns cluster overlays, Flux bootstrap manifests, and application workloads. The infrastructure repository owns provisioning, node bootstrap, and the initial cluster bring-up.

---

## Overview

- Defines the Flux sync target for the Hetzner K3s cluster
- Manages cluster-level Kustomizations
- Manages application workloads through Flux
- Demonstrates Helm-based application delivery with `podinfo`

The current cluster overlay is:

- `./clusters/hetzner-k3s`

Flux tracks:

- path: `./clusters/hetzner-k3s`

---

## Bootstrap Flow

1. The cluster is provisioned by the infrastructure repository.
2. Flux is bootstrapped into the cluster.
3. Flux syncs this repository.
4. Flux applies the cluster overlay under `./clusters/hetzner-k3s`.
5. The overlay reconciles the cluster and application manifests.

This repository is the source of truth for day-2 cluster state.

---

## Scope

This repository manages:

- Cluster overlays
- Flux bootstrap resources
- Application `Kustomization` resources
- Helm repositories and Helm releases

This repository does **not** manage:

- Hetzner Cloud infrastructure
- Node bootstrap
- Tailscale enrollment
- Secret material committed to Git

Those belong in the separate infrastructure repository or in a secret management system.

---

## Repository Layout

```text
.
├── apps
│   ├── kustomization.yaml
│   └── podinfo
│       ├── helmrelease.yaml
│       ├── helmrepository.yaml
│       └── kustomization.yaml
├── clusters
│   └── hetzner-k3s
│       ├── apps.yaml
│       ├── flux-system
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       ├── infrastructure.yaml
│       └── kustomization.yaml
└── README.md
```

---

## Validation

### Repository
Run after changes:

```bash
kustomize build clusters/hetzner-k3s
kubectl apply --dry-run=client -f apps/podinfo/helmrelease.yaml
kubectl apply --dry-run=client -f apps/podinfo/helmrepository.yaml
```

### Cluster
Run against the cluster:

```bash
flux check
flux get sources helm -A
flux get kustomizations -A
flux get helmreleases -A
kubectl get pods -A
```

Pass criteria:

- Flux components are healthy
- the cluster overlay is `Ready=True`
- the application `HelmRelease` is `Ready=True`
- the `podinfo` workload is running

---

## Notes

- Flux is the day-2 reconciliation mechanism.
- `podinfo` is the sample workload in this repository.
- If a `HelmRelease` fails with a missing chart version, the version pin in `apps/podinfo/helmrelease.yaml` is stale.
