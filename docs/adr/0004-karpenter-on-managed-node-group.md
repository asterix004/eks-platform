# ADR-0004: Self-managed Karpenter on a small managed node group

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

The cluster needs to add and remove EC2 capacity automatically as load
changes, and the project should make that mechanism **visible**. Karpenter
watches for pods that cannot be scheduled and launches right-sized nodes for
them, then consolidates (removes or replaces) nodes when they are underused.

Karpenter itself runs as pods, so those pods need somewhere to run that
Karpenter does not manage.

## Options

| Option | Description | Trade-off |
|---|---|---|
| **Managed node group + self-managed Karpenter** | A small AWS-managed node group (2 nodes) runs system pods — Karpenter, CoreDNS, LB controller, Argo CD. Karpenter launches all application nodes. | We install and upgrade Karpenter; every setting (NodePools, disruption budgets, consolidation) is visible and teachable. |
| EKS Auto Mode | AWS runs Karpenter (and core add-ons) for you. | Least operational work; Karpenter internals are hidden, which defeats the showcase. |
| Fargate for system pods | Karpenter runs on Fargate. | Works, but Fargate has its own limitations and adds a third compute model to explain. |
| Cluster Autoscaler | Scales node groups up/down. | Slower, coarser (whole node-group instance types); superseded by Karpenter for this use. |

## Decision

A **managed node group** of 2 small Graviton instances (`t4g.medium`, spread
across AZs, tainted for system workloads) plus **Karpenter installed by
Terraform via Helm**, with:

- a `NodePool` for application pods: on-demand, Graviton (`arm64`), several
  instance families, spread across 3 AZs;
- consolidation enabled, with a disruption budget so consolidation never
  removes too much capacity at once.

## Consequences

- Application images must be built for `arm64` (multi-arch builds in CI).
- Karpenter version upgrades are our responsibility — pinned in Terraform.
- Spot capacity is deliberately **not** used in the failover demo so that a spot
  interruption cannot muddy the results; it may be demonstrated separately.
- Nodes Karpenter launches are **not in Terraform state** — teardown must
  remove them first ([ADR-0012](0012-ephemeral-demo-and-teardown.md)).
