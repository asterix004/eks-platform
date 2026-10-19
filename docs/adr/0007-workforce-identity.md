# ADR-0007: Personas as Identity Center permission sets, managed from the delegated admin account

- **Status:** Accepted
- **Date:** 2026-10-19

## Context

Four human personas need different access:

| Persona | AWS access in the workload account | Kubernetes access (EKS access entries) | Database |
|---|---|---|---|
| **Cloud admin** | AdministratorAccess | `AmazonEKSClusterAdminPolicy` (cluster-wide) | Full (via AWS) |
| **App developer** | ECR push/pull, CloudWatch Logs read, Grafana editor | `AmazonEKSEditPolicy` scoped to app namespaces only | None |
| **DBA** | Aurora/RDS management, Performance Insights, read of the DB master secret only | None | Full |
| **Read-only** | `ViewOnlyAccess` (metadata, not data contents) | `AmazonEKSViewPolicy` | None |

What already exists in the organization (verified 2026-10-19):

- An **organization instance of IAM Identity Center**
  (`ssoins-7223050d04a073a8`) in **us-east-1**.
- Administration is **delegated** to account `821672147486`
  ("Identity Delegated Admin").
- The identity source is **AWS Builder ID**, provisioning "Direct".
- The owner currently signs in through **IAM Account access manager**, which
  creates an AWS-managed role `AccountFullAccessRole` (path `/managed/`,
  trusted only by `account-access.amazonaws.com`) in each account.

## Options

| Option | For | Against |
|---|---|---|
| **Identity Center permission sets + groups** | Mature, fully supported by Terraform (`aws_ssoadmin_*`, `aws_identitystore_*`), clear CloudTrail trail (`AWSReservedSSO_*` roles) | One more layer to explain |
| IAM Account access manager roles | Already in use for the owner's sign-in | New feature; less documentation and Terraform coverage at time of writing |
| Plain IAM roles with MFA | Simple in one account | Doesn't scale; needs IAM users or another identity source |

## Decision

- Model the personas as **Identity Center groups** and **permission sets**,
  assigned to the **workload account** (`093736401430`).
- Terraform for identity runs **in the delegated admin account**
  (`821672147486`), which is where AWS expects Identity Center to be
  administered. (A delegated admin cannot assign permission sets to the
  management account — we don't need it to.)
- Keep the existing **AWS Builder ID** identity source and the owner's current
  sign-in. Changing the identity source would disrupt existing access.
- Map each permission set's role into EKS with **access entries**, not the
  legacy `aws-auth` ConfigMap.

## Open questions

- `[OPEN]` How users are added to groups when the identity source is AWS
  Builder ID (whether `aws_identitystore_user` can create them, or users must
  sign in with a Builder ID first). To be confirmed in Phase 2. Demo persona
  users can be plus-addressed emails (e.g. `<you>+dev@<your-mail-domain>`), each
  created by the owner.
- `[VERIFY]` Amazon Managed Grafana in us-west-2 authenticating users through
  this us-east-1 Identity Center instance from a member account.

## Consequences

- Terraform needs credentials for **three** accounts (management, identity
  delegated admin, workload) — handled with provider aliases and one role per
  account ([ADR-0009](0009-terraform-layering-and-state.md)).
- Each persona's access can be demonstrated: sign in as the persona, try an
  allowed and a denied action, show the CloudTrail record.
