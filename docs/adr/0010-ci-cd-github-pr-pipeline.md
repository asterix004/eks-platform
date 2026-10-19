# ADR-0010: Plan on pull request, apply on merge behind an environment gate

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

Infrastructure changes must be reviewed before they reach AWS, and the review
must show exactly what will change. (GitHub calls these *pull requests*; GitLab
calls them *merge requests*.)

## Decision

Repository: `asterix004/eks-platform`, **public**.

```
PR opened/updated
  ├─ fmt · validate · tflint · checkov · gitleaks
  └─ terraform plan (per changed layer)  ── role: tf-plan (read-only)
        └─ plan posted as a PR comment
Review + CODEOWNERS approval → merge to main
  └─ terraform apply  ── GitHub environment "production" (required reviewer)
                      ── role: tf-apply
        order: permanent layers → us-east-1 → us-west-2
```

**OIDC trust is pinned by the token's `sub` claim:**

- `tf-plan` trusts `repo:asterix004/eks-platform:pull_request`
- `tf-apply` trusts only `repo:asterix004/eks-platform:environment:production`

So a workflow on a fork, another branch, or another repo cannot obtain the
apply role, even if someone edits the workflow file.

## Why public

On GitHub's free plan, **required reviewers on environments** are only
available for public repositories. A public repo containing zero secrets is
also part of the evidence for [ADR-0006](0006-no-static-secrets.md).

## Consequences

- Account IDs and role ARNs are visible in the repo. They are identifiers, not
  credentials; access is controlled by the OIDC trust conditions.
- Pull requests from forks never get AWS credentials (GitHub doesn't issue
  OIDC tokens with our repo's `sub` to forks).
- The plan role must be truly read-only — it can read state and describe
  resources, but not change them.
