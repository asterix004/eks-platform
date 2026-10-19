# ADR-0012: Ephemeral regional stacks with scripted teardown

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

Budget: about **$20 per demo run**. Almost every component is billed hourly,
so the cheapest way to run a production-shaped architecture is to run it only
while it's being demonstrated.

Approximate cost while both regions are up and under load (list prices,
estimates to be checked against the bill after the first run):

| Item | ~$/hour |
|---|---|
| 2 EKS control planes | 0.20 |
| 6 NAT gateways (one per AZ per region) | 0.27 |
| 4 system nodes (t4g.medium) | 0.15 |
| Karpenter nodes at peak (10–15 Graviton, on-demand) | 0.80–1.20 |
| Aurora Serverless v2 global (3 instances) | 0.50–1.50 |
| 2 ALBs | 0.10 |
| Managed Prometheus + collectors | 0.20–0.50 |
| k6 load generator | 0.15 |
| Misc. (health checks, KMS, Secrets Manager, logs) | < 0.10 |
| **Total** | **≈ 2.50–4.00** |

A session (~45–60 min build, ~60 min tests, ~30 min teardown) ≈ **$8–12**.
Monthly fixed costs: Managed Grafana licences (~$9 per editor, ~$5 per
viewer), hosted zone ($0.50), a few cents of CloudTrail storage.

## Decision

- Because we pay by the hour, **don't cut HA corners** (e.g. one NAT gateway);
  build the production shape and destroy it after the demo.
- Regional layers are ephemeral; permanent layers stay ([ADR-0009](0009-terraform-layering-and-state.md)).
- Teardown is a **script**, because some resources are created by controllers
  and are not in Terraform state:
  1. Delete Argo CD applications and Ingresses → wait until the ALBs are gone.
  2. Delete Karpenter NodePools → wait until Karpenter's nodes have terminated.
  3. `terraform destroy` us-west-2, then us-east-1 (global cluster last).
  4. Sweep for anything still tagged `Project=eks-platform` and report it.
- **AWS Budgets alert at $25/month** in the workload account as a safety net.

## Consequences

- Each demo starts from a clean build — which also proves the build is
  reproducible.
- Development iterations cost real money (~$5–10 per full cycle); we build
  region by region to keep cycles short.
