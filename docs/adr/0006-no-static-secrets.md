# ADR-0006: No static secrets anywhere

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

Requirement: secrets must never be hard-coded in GitHub (code, variables or
GitHub Secrets) or in Kubernetes. Long-lived credentials are the most common
cause of cloud breaches; the best secret is one that does not exist.

Every place a credential would traditionally appear, and how we remove it:

| Traditional credential | Replacement | Why it's safe |
|---|---|---|
| AWS access keys in GitHub Secrets | **GitHub OIDC** → `sts:AssumeRoleWithWebIdentity` | GitHub issues a short-lived signed token per job; AWS trusts it only for this repo and specific branches/environments. Nothing to store or leak. |
| DB master password in Terraform / tfvars | `manage_master_user_password = true` | RDS generates and rotates the password in Secrets Manager. Terraform never sees it, so it is **not in state**. |
| DB password for the application | **IAM database authentication** via **EKS Pod Identity** | The pod's IAM role generates a 15-minute auth token. There is no app password at all. |
| Third-party API key (Polygon.io) | **Secrets Manager**, mounted by the **Secrets Store CSI Driver** (AWS provider) as an in-memory file, *without* syncing to a Kubernetes Secret | The value lives only in Secrets Manager and in the memory of the one pod allowed to read it. |
| Grafana → Prometheus API key | **SigV4** using the Grafana workspace's IAM role | AWS request signing; no key. |
| Human passwords to AWS | IAM Identity Center sign-in with MFA; short-lived role sessions | No IAM users, no access keys. |

## Options rejected

- **External Secrets Operator** — popular, but it copies secrets *into*
  Kubernetes Secrets, which violates the requirement.
- **Sealed Secrets / SOPS in Git** — the encrypted secret still lives in Git
  and a decryption key must be managed.

## Decision

Adopt every replacement in the table. The Polygon.io key is the only true
secret in the system: Terraform creates the secret **container** (no value);
the operator sets the value once from the CLI in a way that keeps it out of
shell history. The secret is replicated to us-west-2.

## Consequences / evidence

The evidence report (Phase 8) proves the claim rather than asserting it:

- `gitleaks` scan of the full Git history: clean.
- GitHub repository and environment **secrets lists are empty**.
- `kubectl get secrets` in application namespaces: none created by us.
- Terraform state searched for password/secret values: none.
- CloudTrail shows `GetSecretValue` only from the ingest pod's role, and
  `rds-db:connect` token use from the api pod's role.

Etcd encryption with a KMS key is still enabled on both clusters as defence in
depth.
