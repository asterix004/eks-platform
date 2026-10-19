# eks-platform — a model high-availability platform on AWS

A reference project for running a containerized, three-tier application with
high availability across two AWS regions, built entirely with Terraform and
delivered through GitHub pull requests.

The application is a small **stock dashboard** (about 50 S&P 500 / Nasdaq-100
tickers, end-of-day prices) served from `https://ha.noveon.io`.

## What this project demonstrates

| Capability | How it's shown |
|---|---|
| Elastic capacity | Load test drives pod autoscaling; **Karpenter** adds and removes EC2 nodes |
| Zone resilience | Nodes / an Availability Zone are destroyed with **AWS FIS**; Karpenter replaces capacity in surviving AZs |
| Region resilience | Primary region (us-east-1) is failed; **Route 53** shifts traffic to the warm standby (us-west-2), which scales up and takes over, with **Aurora Global Database** failover |
| No static secrets | GitHub OIDC, RDS-managed DB password, IAM database auth, Secrets Manager mounted without Kubernetes Secrets — with evidence that no secret exists in Git, GitHub, Terraform state or Kubernetes |
| Least-privilege people access | IAM Identity Center permission sets for cloud admin, app developer, DBA and read-only personas, mapped into EKS with access entries |
| GitOps delivery | `terraform plan` on every PR, `terraform apply` on merge behind an approval gate; Argo CD deploys the app |
| Observability | Amazon Managed Service for Prometheus + Amazon Managed Grafana |

## Repository map

```
docs/               ← start here: architecture, decisions, runbooks, concepts, build journal
terraform/          ← infrastructure (bootstrap, permanent and per-region layers)   [coming]
app/                ← FastAPI web tier, api tier and ingest job                     [coming]
deploy/             ← Helm charts, Argo CD apps, Karpenter NodePools                [coming]
load/  chaos/       ← k6 load tests, FIS experiments                                [coming]
.github/workflows/  ← Terraform PR pipeline, app image builds                       [coming]
```

## Reading order

1. [docs/README.md](docs/README.md) — how the documentation is organized
2. [docs/architecture/overview.md](docs/architecture/overview.md) — the big picture
3. [docs/adr/](docs/adr/README.md) — why each design choice was made
4. [docs/runbooks/](docs/runbooks/) — how to build it, step by step, with verification

## Cost

Designed to be **ephemeral**: the regional stacks are created for a demo
session (~2.5–3 hours, roughly $10–12) and destroyed by a scripted teardown.
See [ADR-0012](docs/adr/0012-ephemeral-demo-and-teardown.md).
