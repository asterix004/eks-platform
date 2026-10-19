# ADR-0002: us-east-1 primary, us-west-2 secondary

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

The original brief named `us-east1` and `us-central1`; those are Google Cloud
region names. AWS has no "central" US region. We need two AWS regions that make
a credible disaster-recovery story and support every service we use (EKS,
Karpenter, Aurora Global Database, Managed Prometheus, Managed Grafana, FIS).

## Options

| Pair | For | Against |
|---|---|---|
| **us-east-1 + us-west-2** | ~4,000 km apart: separate power grids, weather and seismic zones. Both are AWS's largest regions with every service. | Higher cross-region latency (~60–70 ms); irrelevant for asynchronous Aurora replication. |
| us-east-1 + us-east-2 | Closest to "east + central"; lower latency. | Both in the eastern US; a weaker story for a regional-disaster demo. |

## Decision

**us-east-1 (N. Virginia) is primary; us-west-2 (Oregon) is the warm standby.**

## Consequences

- IAM Identity Center for this organization already lives in **us-east-1**
  (see [ADR-0007](0007-workforce-identity.md)). A real us-east-1 outage could
  affect sign-in; our failover drill is simulated, so the demo is unaffected,
  but the runbooks document an emergency-access path.
- Amazon Managed Grafana is placed in **us-west-2** so dashboards stay
  reachable during the primary-region drill ([ADR-0013](0013-observability-amp-amg.md)).
- Cross-region data transfer (Aurora replication, ECR image replication) is
  billed but small at demo volumes.
