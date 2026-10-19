# Concepts

Plain-language explainers of the mechanisms this project relies on. Each one is
written when we first build the piece it explains.

| Concept | Written in phase |
|---|---|
| AWS Organizations, Identity Center and the accounts in this project | 0 |
| How GitHub Actions gets AWS credentials without a secret (OIDC trust) | 1 |
| Terraform state, locking and why the layers are split | 1 |
| Permission sets, groups and EKS access entries — how a person gets into a cluster | 2 |
| How Karpenter decides which node to launch, and when it removes nodes | 3 |
| EKS Pod Identity — how a pod gets an IAM role | 3 |
| IAM database authentication — a database login without a password | 4 |
| How Aurora Global Database replicates and fails over | 5 |
| How Route 53 health checks detect an outage and fail over | 5 |
| How Prometheus metrics get from a pod to a Grafana panel | 6 |
| Autoscaling chain: request rate → pods → pending pods → nodes | 7 |
| AWS FIS experiments and stop conditions | 8 |
