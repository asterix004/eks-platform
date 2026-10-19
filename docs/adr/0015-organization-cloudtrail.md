# ADR-0015: Permanent organization CloudTrail, created by Terraform

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

CloudTrail **Event history** already shows 90 days of management events per
account, per region, for free, without any setup. But it can't be searched
across accounts, it expires, and CloudTrail trusted access in the organization
is not enabled yet.

The console offers a "Quick trail create" with an auto-named bucket. We did
**not** use it: it would create an untracked single-account trail outside
Terraform.

## Decision

- An **organization trail** created by Terraform in the management account
  (`permanent/org` layer), covering all accounts and all regions.
- Logs go to an S3 bucket with SSE-KMS, public access blocked, log file
  validation on, and a lifecycle rule that deletes logs after 90 days.
- Management events only (the first copy is free); no data events.
- This trail is **kept** when the demo is torn down. It is account hygiene,
  not demo infrastructure, and it holds evidence: `GetSecretValue` calls,
  pipeline role sessions, persona sign-ins, FIS actions.

## Consequences

- Cost: S3 storage for a few MB/month — cents.
- It will be removed only if the whole project is retired.
