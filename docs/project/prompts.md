# Project prompts log

The owner's instructions that shaped this project, in order, with the outcome
of each. Prompts are quoted as written, except that email addresses are
replaced with `<email>` because this repository is public. Screenshots that
accompanied some prompts are described in brackets.

## Session 1 — 2026-10-18 / 19

### 1. The brief

> Please create a project showcasing a high availability application hosted on a kubernetes cluster leveraging EKS and Karpenter. The entire stack should be created by Terraform including the necessary IAM roles for various users - app developers, database admins, a read only user and an overall cloud admin role. also showcase how this Terraform can be applied via a GitHUB merge request so the pipelines will show the terraform apply process as well. Next leverage Grafana and Prometheus - Managed Grafana in AWS. Also showcase how secrets are managed without ever hardcoding these in any GitHub variables or Kubernetes. We can choose us-east1 and us-central1 as the two regional sites with a regional ALB. This will be a classic three tier architecture - the presentation layer, the app layer, the database layer. We will create a load that will generate more and more requests until EKS spins more nodes to ensure requests are honored. Next we terminate some nodes in one specific region to check if EKS can spin up a set of nodes in the secondary region to ensure business continuity...
> Before you build anything lets brainstorm on how this can be a model project for containerized workloads that need high availability on AWS.
> Stack is in AWS - Leverage EKS, control plane and node group managed by AWS and IaC in Terraform. Secrets OIDC for GitHUB, other details as we chat

**Outcome:** brainstorm only. Corrections raised: `us-east1`/`us-central1` are
Google Cloud names; an EKS cluster cannot place nodes in another region (so:
one cluster per region + global traffic steering); a regional ALB needs a
global layer above it. Decisions taken via questions:
us-east-1 + us-west-2 ([ADR-0002](../adr/0002-regions.md)), warm standby
([ADR-0003](../adr/0003-multi-region-warm-standby.md)), IAM Identity Center
([ADR-0007](../adr/0007-workforce-identity.md)), Argo CD
([ADR-0011](../adr/0011-argo-cd-app-delivery.md)).

### 2. Budget

> Budget is enough for a demo - max we will need to see the failover and the secondary region pick the load up - that is the essential test and also evidence that secrets are never exposed, karpenter helps scale the nodes etc. - I am thinking the demo budget is something like $20 for an hour worth of traffic at the most - we tear things down once our tests are completed? what do you think

**Outcome:** $20 is realistic for the demo run if regional stacks are
ephemeral and teardown is scripted; development iterations will cost more.
Three essential proofs defined. See
[ADR-0012](../adr/0012-ephemeral-demo-and-teardown.md).

### 3. Domain, account, documentation, language

> We can use noveon.io (i own this) - I also own an AWS account (093736401430 with `<email>` - also see the snippet of the management account "Try Catch Finally") as root)but nothing is set up, we will need Identity Center enabled and/or an organization. Please ensure this is a project that I want to understand the details of how you build what I requested so documentation of processes is crucial. Lets use Python/Fast API as our stack

[Screenshot: console account menu for 093736401430, signed in as
`AccountFullAccessRole`.]

**Outcome:** documentation became a first-class deliverable
([docs/README.md](../README.md)). noveon.io DNS found at Namecheap →
delegate a subdomain ([ADR-0014](../adr/0014-dns-subdomain-delegation.md)).
Found a second CLI profile pointing at another account.

### 4. Accounts and subdomain

> I've run aws login for both profiles. Inspect both accounts read-only. 223703744147 should be the management account. Use ha.noveon.io and a public repo asterix004/eks-platform.

> ok i was debating on ha.noveon.io - its fine lets use that - it would be cool to just show a small stock app as the application (serving a daily snippet of some 50 select stocks in S&P and Nasdaq just as an example)

**Outcome:** `ha.noveon.io`, public repo, stock dashboard application
([ADR-0016](../adr/0016-stock-app-and-market-data.md)).

### 5. Market-data key

> logged in - first logged into the mgmt account (223703744147) and then into the workload account (093736401430). Created an account in polygon.io with `<email>`. Do you need the API key or should I store it in a file that can be accessed later?

**Outcome:** neither — the key stays in the owner's password manager and goes
straight into Secrets Manager in Phase 2
([ADR-0006](../adr/0006-no-static-secrets.md)).

### 6. CLI sign-in and root

> 1 browser window opened for the mgmt account and i clicked that and it went away - should i expect the second one

> hit n for both

> help me check it carefully - maybe we are missing something

> what is my cli login , i should not be using roiot!

**Outcome:** `aws login` had offered to save **root** credentials (declined).
Established the rule: never use root for the CLI or daily work.

### 7. Investigating the existing setup

[Screenshots: `AccountFullAccessRole` trust policy and permissions;
Organizations account list; CloudTrail quick-create page.]

> question- cloudtrail s3 bucket - will we delete this at the end of the demo

[Screenshots: Identity Center (configured in us-east-1); Organizations
trusted services; IAM dashboard showing root MFA missing.]

[Screenshots: root MFA assigned; Identity Center settings — delegated admin,
AWS Builder ID source, portal URL; Root access management showing "Access
denied" for member accounts.]

**Outcome:** an Organization, a delegated Identity Center and an AWS-managed
access role already existed; root MFA was added. Quick-create trail cancelled
in favour of a permanent Terraform organization trail
([ADR-0015](../adr/0015-organization-cloudtrail.md)). Findings recorded in the
[Phase 0 journal](../journal/2026-10-19-phase-0-discovery.md).

### 8. Publishing and wrapping up

> Commit the docs, create the public repo asterix004/eks-platform and push main.

> I would like to stop for the day, have a 8:00am work meeting
> before we quit should i log off from all sesssions that i have root

> Please save the prompts for this project and perhaps rewrite the project and the expectations from this so that we can start a project doc

**Outcome:** this prompts log, plus a project charter document summarising
purpose, goals, scope, decisions and status.
