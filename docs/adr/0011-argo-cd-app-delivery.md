# ADR-0011: Argo CD per cluster deploys the application

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

Infrastructure changes (Terraform) and application releases have different
cadences and owners. App developers should be able to release without
touching Terraform or holding cluster-admin credentials.

## Options

| Option | For | Against |
|---|---|---|
| **Argo CD (GitOps)** | Cluster *pulls* desired state from Git; drift is shown and corrected; no cluster credentials in CI | One more component to run |
| GitHub Actions + `helm upgrade` | One tool | CI needs credentials into both clusters; no drift detection |

## Decision

- Terraform installs **Argo CD** in each cluster as a platform add-on.
- Each Argo CD watches `deploy/` in this repo and deploys the Helm charts with
  per-region values.
- CI builds multi-arch images, pushes to ECR (replicated to us-west-2), and
  updates the image tag in Git via a pull request. Merging releases to both
  regions.

## Consequences

- Clear boundary: Terraform owns the platform, Argo CD owns workloads.
- Argo CD is reached via `kubectl port-forward` (no public UI) to keep the
  attack surface small.
- During teardown, Argo CD applications are deleted first so that the ALBs
  they created are removed cleanly ([ADR-0012](0012-ephemeral-demo-and-teardown.md)).
