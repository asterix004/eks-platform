# ADR-0008: Management / identity / workload account separation

- **Status:** Accepted
- **Date:** 2026-10-19

## Context

An AWS Organization (`o-slswscsay1`) already exists with three accounts:

| Account | ID | Purpose in this project |
|---|---|---|
| menon-noveon-team Management Account | `223703744147` | Organization, billing, organization CloudTrail. **No workloads.** |
| menon-noveon-team Identity Delegated Admin | `821672147486` | Administers IAM Identity Center (permission sets, groups, assignments) |
| Try Catch Finally | `093736401430` | **Workload account**: everything this project runs — EKS, Aurora, Route 53 zone, ECR, AMP, Grafana, FIS |

## Decision

Keep this separation. Nothing runs in the management account except
organization-level resources that must live there.

## Why

- The management account cannot be restricted by Service Control Policies; a
  compromise there affects every account. Keeping it nearly empty shrinks that
  risk.
- A separate workload account gives a clean **blast radius** and a clean
  **cost boundary**: the demo's spend is exactly that account's bill.
- Identity administration delegated away from the management account means
  day-to-day identity work never needs management-account access.

## Consequences

- Cross-account Terraform: providers per account (see ADR-0009).
- Root user hygiene per account: MFA on the management root (done
  2026-10-18); member-account root credentials to be removed with
  **centralized root access management** (Phase 0 step, see runbook 00).
