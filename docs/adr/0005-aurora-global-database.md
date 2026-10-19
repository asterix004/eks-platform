# ADR-0005: Aurora PostgreSQL Global Database, Serverless v2

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

The data tier must survive a regional failure with minimal data loss and be
recognisably "classic three-tier" — a relational database with a DBA persona
who manages it.

## Options

| Option | Behaviour | Fit |
|---|---|---|
| **Aurora PostgreSQL Global Database** | One writer region; storage-level replication to the secondary, typically under 1 s lag; managed switchover (planned) and failover (unplanned); a global writer endpoint that follows the writer | Classic relational tier, well-understood failover |
| Aurora DSQL | Writes accepted in multiple regions | Not full PostgreSQL; different concurrency model; less "classic" |
| DynamoDB global tables | Multi-region writes | Not relational; no DBA story |
| RDS PostgreSQL + cross-region read replica | Simple | Manual promotion, slower recovery |

## Decision

**Aurora PostgreSQL Global Database** with **Serverless v2** instances:

- Primary cluster in us-east-1: writer + one reader in a different AZ (fast
  in-region failover).
- Secondary cluster in us-west-2: one reader (becomes writer on failover).
- Capacity range 0.5–8 ACU so it costs little while idle and grows under load.
- Master password managed by RDS in Secrets Manager; the application uses
  **IAM database authentication** ([ADR-0006](0006-no-static-secrets.md)).

## Consequences

- Region failover is an explicit operation (`failover-global-cluster`), run by
  the drill in Phase 8. Expected recovery in the order of a minute; data loss
  bounded by replication lag. Both are measured, not assumed.
- Applications in the secondary region read from the local reader; nothing
  connects across regions.
- The ingest job writes only where the writer is — it checks before writing
  ([ADR-0016](0016-stock-app-and-market-data.md)).
- Deletion protection and final snapshots are **off** for the ephemeral demo
  and are variables, so a production use flips them on.
