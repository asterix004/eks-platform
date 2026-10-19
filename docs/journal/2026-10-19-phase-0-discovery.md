# 2026-10-18/19 — Phase 0: discovering what already existed

## Starting assumption

"One AWS account, nothing set up." The plan was to create an Organization and
enable IAM Identity Center.

## What we actually found

1. **Two CLI profiles pointed at two different accounts.** `noveon` →
   `093736401430`, `noveon-mgmt` → `223703744147`, both signed in through a
   role named `AccountFullAccessRole` with the *same* session name. Same
   identity, two accounts: something was already federating access.

2. **The CLI sessions had expired, and `aws login` offered to save root.**
   `aws login` borrows the browser's console session. The browser was signed
   in as the management root user, so the CLI offered
   `arn:aws:iam::223703744147:root`. We declined.
   *Lesson:* check which identity the browser holds before running
   `aws login`.

3. **`AccountFullAccessRole` is AWS-managed, not custom.** Its path is
   `/managed/`, its only trusted principal is the service
   `account-access.amazonaws.com`, and it has `AdministratorAccess`. It is
   created by **IAM Account access manager**, which the organization had
   enabled.

4. **An Organization already existed** (`o-slswscsay1`, created 2026-10-05)
   with a third account we didn't know about: **Identity Delegated Admin**
   (`821672147486`).

5. **IAM Identity Center was already enabled** — organization instance in
   us-east-1, administration delegated to `821672147486`, identity source
   **AWS Builder ID**. The console showed only a landing page at first
   because we were looking from the Ohio region; Identity Center has a single
   home region.

6. **The management root user had no MFA.** Fixed on 2026-10-18 (virtual MFA
   device). Root had no access keys.

7. **CloudTrail:** no trail. The console offered "Quick trail create"; we
   cancelled it in favour of a Terraform-managed organization trail
   ([ADR-0015](../adr/0015-organization-cloudtrail.md)). Event history (90
   days, free) was enough for the investigation.

8. **Root access management** showed "Access denied" for member-account root
   credential status while viewed as root. Still open (runbook 00, step 0.3).

## Decisions this changed

- No Organization or Identity Center to create — reuse them.
- Identity Center stays in us-east-1 (moving it means recreating it);
  consequence recorded in [ADR-0002](../adr/0002-regions.md).
- Personas will be managed from the delegated admin account
  ([ADR-0007](../adr/0007-workforce-identity.md)) — so Terraform needs a third
  account's credentials.
- Account roles recorded in [ADR-0008](../adr/0008-account-structure.md).

## Takeaway

Inspect before you build. Every one of these findings would have caused a
failed or harmful Terraform apply had we assumed an empty account: creating a
second Organization isn't possible for a member account, and creating
permission sets from the wrong account fails.
