# ADR-0013: Managed Prometheus, managed collectors and Managed Grafana

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

We need to *watch* the demo: request rates, latency and errors per region,
pod and node counts, Karpenter provisioning, Aurora replication lag. We don't
want to operate Prometheus servers inside the clusters we are deliberately
breaking.

## Decision

- **Amazon Managed Service for Prometheus (AMP):** one workspace per region.
- **AMP managed collectors** (agentless scrapers run by AWS) scrape each
  cluster's metrics — no Prometheus server or agent pods to run.
- **Amazon Managed Grafana (AMG):** one workspace in **us-west-2**, so
  dashboards stay up during the primary-region drill. Data sources: both AMP
  workspaces and CloudWatch (ALB, Aurora, Route 53 health checks), using
  SigV4 with the workspace's IAM role — no API keys.
- Users sign in to Grafana through IAM Identity Center; persona groups map to
  Grafana roles (admin / editor / viewer).
- Dashboards are stored as JSON in this repo and provisioned by Terraform.

## Open questions

- `[VERIFY]` AMG workspace in us-west-2, in a member account, using the
  organization's Identity Center in us-east-1. If unsupported, fall back to
  us-east-1 and document the trade-off.

## Consequences

- Metrics ingestion is billed per sample; scrape intervals are tuned (30 s) to
  keep cost low.
- A regional AMP outage loses that region's metrics — acceptable for a demo,
  and the drill doesn't actually take AWS services down.
