# ADR-0009: Terraform split into bootstrap, permanent and per-region layers

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

One giant Terraform state would mean: every plan touches everything, a failure
in one region blocks changes to the other, and teardown can't keep the cheap
long-lived parts. The design also has a chicken-and-egg problem — the pipeline
needs a state bucket and an AWS role that must exist *before* the pipeline can
run.

## Decision

| Layer | Contents | Applied by | Lifetime |
|---|---|---|---|
| `bootstrap` | State bucket + KMS key, GitHub OIDC provider, `tf-plan` / `tf-apply` roles (one per account) | Operator, once, from a laptop | Permanent |
| `permanent/identity` | Identity Center groups, permission sets, assignments | Pipeline | Permanent |
| `permanent/org` | Organization CloudTrail | Pipeline | Permanent |
| `permanent/global` | Route 53 zone `ha.noveon.io`, ECR repos + replication, Secrets Manager secret container, Grafana workspace, budgets | Pipeline | Permanent (a few $/month) |
| `regional/us-east-1` | VPC, EKS, Karpenter, add-ons, Aurora primary, AMP, ALB DNS records, FIS templates | Pipeline | **Ephemeral** |
| `regional/us-west-2` | Same, Aurora secondary | Pipeline | **Ephemeral** |

- **State backend:** one S3 bucket in the workload account, versioned,
  encrypted with a KMS key, public access blocked, with **S3 native locking**
  (`use_lockfile = true`, Terraform ≥ 1.10) — no DynamoDB table needed.
- One state file per layer (`<layer>/terraform.tfstate`).
- Layers read each other's outputs through `terraform_remote_state` or SSM
  parameters, never by hard-coding.
- Shared logic lives in `terraform/modules/`; both regions use the same
  modules with different inputs.

## Consequences

- Region B can be planned and applied while region A is down.
- Teardown destroys only the regional layers.
- The bootstrap layer is the single manual step and is documented in runbook 01.
- Provider versions and Terraform version are pinned.
