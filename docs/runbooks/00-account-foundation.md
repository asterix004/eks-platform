# Runbook 00 — Account foundation

**Goal:** a secure, understood starting point before any Terraform runs:
the right accounts, no root usage, working CLI access to every account
Terraform will touch, and local tools installed.

**Who:** the account owner, in the AWS console and a terminal.
**Cost:** $0.

## Background: what was already in place

Discovered 2026-10-18/19 (see the [journal entry](../journal/2026-10-19-phase-0-discovery.md)):

| Item | State |
|---|---|
| AWS Organization `o-slswscsay1` | Exists (created 2026-10-05) |
| Accounts | Management `223703744147`, Identity Delegated Admin `821672147486`, Workload "Try Catch Finally" `093736401430` |
| Trusted access enabled | IAM Identity Center, Account access manager, IAM (root access management), Account Management, User Notifications |
| Trusted access **not** enabled | CloudTrail (handled by Terraform in Phase 2) |
| IAM Identity Center | Organization instance `ssoins-7223050d04a073a8`, **us-east-1**, delegated admin `821672147486`, identity source **AWS Builder ID** |
| Owner's sign-in | Through **Account access manager** → AWS-managed role `AccountFullAccessRole` (AdministratorAccess) in each account |
| IAM users / access keys | None (good) |

---

## Step 0.1 — Secure the management account's root user ✅ done 2026-10-18

**What:** add MFA to the root user of `223703744147`.

**Why:** the management root user can do anything to the organization,
including closing member accounts, and cannot be restricted by any policy.
Without MFA, its email + password are the only protection.

**Verify:** IAM → Dashboard → *Security recommendations* shows no red badge;
*My security credentials* (as root) lists one MFA device and **0 access keys**.

## Step 0.2 — Stop using root

**What:** sign out of root. From now on use the `AccountFullAccessRole`
sessions (and, after Phase 2, Identity Center permission sets).

**Why:** root has no guardrails and its actions are harder to attribute.
Only a short list of tasks requires root (closing an account, changing the
account email or support plan, restoring locked-out access).

**Watch out:** the CLI `aws login` command signs the CLI in **as whatever
identity the browser is signed in as**. If the browser is on a root session,
the CLI offers to save root credentials. **Always answer `n`**, switch the
browser to an `AccountFullAccessRole` session, and run it again.

**Break-glass:** if IAM Identity Center or us-east-1 is unavailable, the
management root user (with its MFA device) is the emergency path. Store the
root password and MFA recovery details offline.

## Step 0.3 — Centralized root access for member accounts `[OPEN]`

**What:** IAM → *Root access management* in the management account. Enable
**Root credentials management** and **Privileged root actions in member
accounts**, then optionally **delete the root credentials** of
`821672147486` and `093736401430`.

**Why:** member accounts were created by Organizations, so their root users
have never been used — but a root password can still be reset by anyone who
controls the account's email. Removing member root credentials leaves only
one root user in the organization (the management one, with MFA). Rare root
tasks in member accounts are then done as short, audited *privileged
sessions* from the management account.

**Status:** viewing the page while signed in as **root** showed
"Access denied" in the *Root user credentials* column for both member
accounts. Retry while signed in with the `AccountFullAccessRole` session in
the management account.

**Verify:** the *Root user credentials* column shows "Not present" for both
member accounts.

> This is a security setting: decide it yourself. It's recommended but not
> required for the demo.

## Step 0.4 — CLI access to all three accounts

Terraform needs credentials for three accounts. Configure one CLI profile per
account, each using the `AccountFullAccessRole` session (never root):

| Profile | Account | Used for |
|---|---|---|
| `noveon-mgmt` | `223703744147` | Organization CloudTrail |
| `noveon-identity` | `821672147486` | Identity Center personas |
| `noveon` | `093736401430` | Everything else |

```bash
aws configure set region us-east-1 --profile noveon-identity
```

Then, with the browser switched to the matching `AccountFullAccessRole`
session each time:

```bash
aws login --profile noveon-mgmt
```
```bash
aws login --profile noveon-identity
```
```bash
aws login --profile noveon
```

**Verify:** each returns an `assumed-role/AccountFullAccessRole/...` ARN in the
right account — **never** `:root`.

```bash
for p in noveon-mgmt noveon-identity noveon; do aws sts get-caller-identity --profile $p --query Arn --output text; done
```

**Later (Phase 2):** these profiles are replaced by Identity Center profiles
(`aws configure sso`, start URL
`https://ssoins-7223050d04a073a8.portal.us-east-1.app.aws`) using the
persona permission sets.

## Step 0.5 — Market-data API key

**What:** keep the Polygon.io API key in your password manager. Do **not**
paste it into chat, a file, an environment file, or this repo.

**Why:** it is the project's only external secret
([ADR-0006](../adr/0006-no-static-secrets.md)). You'll store it directly
into Secrets Manager in Phase 2 using a command that reads it without echoing
or recording it in shell history.

## Step 0.6 — Local tools

| Tool | Required | Installed (2026-10-18) |
|---|---|---|
| Terraform | ≥ 1.10 (S3 native locking) | 1.16.5 ✅ |
| AWS CLI | v2 with `aws login` | 2.37.13 ✅ |
| kubectl | within one minor version of EKS | 1.37.1 ✅ |
| Helm | 3 or 4 | 4.3.0 ✅ |
| Docker | buildx for multi-arch images | 29.9.0 ✅ |
| GitHub CLI | authenticated | 2.102.0, `asterix004` ✅ |
| uv (Python 3.13) | yes | ❌ install: `brew install uv` |
| k6 | optional (runs in AWS) | ❌ |

## Exit criteria

- [x] Management root has MFA, no access keys
- [ ] Signed out of root; browser uses `AccountFullAccessRole` sessions
- [ ] Decision taken on centralized root access (Step 0.3)
- [ ] `sts get-caller-identity` succeeds for all three profiles, none as root
- [ ] `uv` installed
