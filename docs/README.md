# Documentation guide

This project is meant to be **understood**, not just run. Every piece of
infrastructure is accompanied by an explanation of what it is, why it was
chosen, and how to prove it works.

## How the docs are organized

| Folder | Answers | Style |
|---|---|---|
| [`architecture/`](architecture/overview.md) | *What does the whole system look like?* | Diagrams + narrative |
| [`adr/`](adr/README.md) | *Why was it designed this way?* | One Architecture Decision Record per decision: context, options, decision, consequences |
| [`runbooks/`](runbooks/) | *How do I build / operate it?* | Numbered phases. Every step says what it does, why, and how to **verify** it |
| [`concepts/`](concepts/README.md) | *How does this mechanism actually work?* | Plain-language explainers (OIDC trust, Karpenter, IAM DB auth, Route 53 failover…) |
| [`journal/`](journal/) | *What happened when we built it?* | Dated build log: what we did, what we found, what broke and how it was fixed |
| [`project/`](project/prompts.md) | *What was asked for?* | The owner's original prompts, in order, with the outcome of each |

## Conventions

- **Verify sections** list commands with their expected output. A step is not
  done until its verification passes.
- **`[OPEN]`** marks a question not yet answered. Open items are tracked in the
  runbook where they block progress.
- **`[VERIFY]`** marks a claim about AWS behaviour that we will confirm when we
  reach that phase, rather than assume.
- Account IDs, ARNs and URLs in these docs are identifiers, **not secrets**.
  Nothing in this repository is a credential.

## Build phases

| # | Phase | Runbook |
|---|---|---|
| 0 | Account foundation (console, one-time) | [00-account-foundation.md](runbooks/00-account-foundation.md) |
| 1 | Bootstrap: Terraform state, GitHub OIDC, pipeline roles | *coming* |
| 2 | Permanent layer: personas, DNS, ECR, Grafana, CloudTrail | *coming* |
| 3 | Primary region: VPC, EKS, Karpenter, Argo CD, Aurora, AMP | *coming* |
| 4 | Application: web, api, ingest | *coming* |
| 5 | Secondary region + global failover | *coming* |
| 6 | Dashboards | *coming* |
| 7 | Load test & Karpenter scaling | *coming* |
| 8 | Chaos, failover drill & evidence report | *coming* |
| 9 | Teardown | *coming* |
