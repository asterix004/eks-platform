# Architecture Decision Records

An ADR captures one significant decision: the situation that forced it, the
options considered, what was chosen, and what we accept as a consequence.
ADRs are never edited to change a decision — a new ADR **supersedes** the old
one, so the history of our thinking stays visible.

Status values: `Proposed` · `Accepted` · `Superseded by ADR-NNNN`

| ADR | Decision | Status |
|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record decisions as ADRs | Accepted |
| [0002](0002-regions.md) | us-east-1 primary, us-west-2 secondary | Accepted |
| [0003](0003-multi-region-warm-standby.md) | One EKS cluster per region, warm standby, Route 53 failover | Accepted |
| [0004](0004-karpenter-on-managed-node-group.md) | Self-managed Karpenter on a small managed node group (not EKS Auto Mode) | Accepted |
| [0005](0005-aurora-global-database.md) | Aurora PostgreSQL Global Database, Serverless v2 | Accepted |
| [0006](0006-no-static-secrets.md) | No static secrets anywhere | Accepted |
| [0007](0007-workforce-identity.md) | Personas as Identity Center permission sets, managed from the delegated admin account | Accepted |
| [0008](0008-account-structure.md) | Management / identity / workload account separation | Accepted |
| [0009](0009-terraform-layering-and-state.md) | Terraform split into bootstrap, permanent and per-region layers | Accepted |
| [0010](0010-ci-cd-github-pr-pipeline.md) | Plan on PR, apply on merge behind an environment gate; public repo | Accepted |
| [0011](0011-argo-cd-app-delivery.md) | Argo CD per cluster deploys the application | Accepted |
| [0012](0012-ephemeral-demo-and-teardown.md) | Ephemeral regional stacks with scripted teardown | Accepted |
| [0013](0013-observability-amp-amg.md) | Managed Prometheus + managed collectors + Managed Grafana | Accepted |
| [0014](0014-dns-subdomain-delegation.md) | Delegate `ha.noveon.io` from Namecheap to Route 53 | Accepted |
| [0015](0015-organization-cloudtrail.md) | Permanent organization CloudTrail, created by Terraform | Accepted |
| [0016](0016-stock-app-and-market-data.md) | FastAPI stock dashboard fed by Polygon.io end-of-day data | Accepted |
