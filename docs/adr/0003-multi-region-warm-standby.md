# ADR-0003: One EKS cluster per region, warm standby, Route 53 failover

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

The brief asked to "terminate nodes in one region and have EKS spin up nodes in
the secondary region." That is not how EKS works: an EKS cluster — and the
Karpenter running inside it — exists in **one region**. It can spread nodes
across that region's Availability Zones, never into another region.

Resilience therefore has two layers that must be designed separately:

1. **Within a region** — survive the loss of nodes or a whole Availability Zone.
2. **Across regions** — survive the loss of a whole region.

## Options

| Topology | Description | Trade-off |
|---|---|---|
| Active-active | Both regions serve users; writes forwarded to the single Aurora writer | Most impressive; adds write-forwarding latency and makes failover less visible |
| **Warm standby** | Region B runs the full stack at minimum size and receives traffic only on failover | Clearest business-continuity story: you *see* region B scale up |
| Pilot light / backup-restore | Region B has data only; compute created on failover | Cheapest; recovery takes tens of minutes |

Global traffic steering options: **Route 53 failover records with health
checks** (DNS-based, simple, cheap) or AWS Global Accelerator (anycast IPs,
faster failover, ~$18/month fixed fee plus data).

## Decision

- **Two independent EKS clusters**, one per region, each spread across three
  Availability Zones.
- **Warm standby:** us-west-2 runs every component at minimum replicas.
- **Route 53 failover routing** on `ha.noveon.io`, with health checks against
  each region's ALB. Each region has its own ALB, created by the AWS Load
  Balancer Controller.

## Consequences

- The "nodes move to another region" demo becomes two clearer demos: AZ loss
  (Karpenter refills capacity in other AZs) and region loss (Route 53 moves
  traffic; Karpenter in us-west-2 scales up to absorb it).
- Failover time is bounded by health-check detection (~30 s with fast checks)
  plus client DNS caching (record TTL 60 s) — measured in Phase 8.
- Everything regional must be deployable **independently**: region B must be
  operable while region A is down ([ADR-0009](0009-terraform-layering-and-state.md)).
- No network connection between the regions is needed or built.
