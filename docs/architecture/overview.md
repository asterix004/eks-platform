# Architecture overview

## In one paragraph

A three-tier stock dashboard runs on **two independent EKS clusters**, one in
**us-east-1** (primary) and one in **us-west-2** (warm standby). Each cluster
spreads across three Availability Zones, and **Karpenter** adds and removes
nodes as load changes. Data lives in an **Aurora PostgreSQL Global Database**
whose writer is in us-east-1, replicated to a reader in us-west-2.
**Route 53** sends all users to us-east-1 while it is healthy and to us-west-2
when it isn't. Everything is built by **Terraform** through GitHub pull
requests, with **no static secrets** anywhere.

## Picture

```mermaid
flowchart TB
    user([Users / k6 load]) --> r53{{"Route 53<br/>ha.noveon.io<br/>failover + health checks"}}

    subgraph east["us-east-1 — PRIMARY"]
        albE[ALB] --> webE["web pods<br/>(FastAPI + HTMX)"]
        webE --> apiE["api pods<br/>(FastAPI)"]
        apiE --> auroraE[("Aurora writer<br/>+ reader")]
        ingE["ingest CronJob"] --> auroraE
        karpE["Karpenter"] -. launches nodes .-> webE
        ampE[(AMP)]
    end

    subgraph west["us-west-2 — WARM STANDBY"]
        albW[ALB] --> webW["web pods<br/>(min replicas)"]
        webW --> apiW["api pods<br/>(min replicas)"]
        apiW --> auroraW[("Aurora reader<br/>→ writer on failover")]
        ingW["ingest CronJob<br/>(no-op unless writer)"] --> auroraW
        karpW["Karpenter"] -. launches nodes .-> webW
        ampW[(AMP)]
        amg["Managed Grafana"]
    end

    r53 -- primary --> albE
    r53 -. secondary .-> albW
    auroraE == "storage replication (< 1 s)" ==> auroraW
    amg --> ampE
    amg --> ampW
    sm[("Secrets Manager<br/>Polygon.io key<br/>(replicated)")] -.-> ingE
    sm -.-> ingW
```

## The three tiers

| Tier | What runs | Scales by |
|---|---|---|
| Presentation | `web` — server-rendered pages (FastAPI, Jinja2, HTMX) | Pod autoscaling on CPU / request rate; Karpenter adds nodes |
| Application | `api` — JSON API computing indicators per request | Same |
| Data | Aurora PostgreSQL Serverless v2 (Global Database) | Aurora capacity units (0.5–8 ACU) |

## Inside one region

```mermaid
flowchart LR
    subgraph vpc["VPC — 3 Availability Zones"]
        direction TB
        pub["Public subnets<br/>ALB, NAT gateways"]
        priv["Private subnets<br/>EKS nodes and pods"]
        db["Database subnets<br/>Aurora (no internet route)"]
        pub --> priv --> db
    end
    subgraph eks["EKS cluster"]
        mng["Managed node group (2 × t4g.medium)<br/>Karpenter · CoreDNS · LB controller · Argo CD · CSI drivers"]
        kp["Karpenter NodePool<br/>Graviton on-demand, 3 AZs<br/>web · api · ingest pods"]
    end
```

## The accounts

| Account | ID | Holds |
|---|---|---|
| Management | `223703744147` | Organization, billing, organization CloudTrail |
| Identity Delegated Admin | `821672147486` | IAM Identity Center administration (personas) |
| Workload — "Try Catch Finally" | `093736401430` | Everything in the diagrams above |

## Failure scenarios the demo exercises

| Scenario | What happens | What we watch |
|---|---|---|
| Load rises | Pod autoscaler adds pods → pods can't fit → Karpenter launches nodes in ~1 min | Pending pods, node count, request latency |
| Load falls | Pods scale in → Karpenter consolidates and removes nodes | Node count shrinking, cost per hour |
| Nodes terminated (FIS) | Pods rescheduled; Karpenter replaces capacity | Error rate stays low thanks to PodDisruptionBudgets and spread |
| AZ power interruption (FIS) | Capacity is rebuilt in the two remaining AZs | Same, plus Aurora in-region failover if the writer was in that AZ |
| Region failure (drill) | Health checks fail → Route 53 moves traffic to us-west-2 → Aurora global failover → Karpenter in us-west-2 scales up | The page banner flips from us-east-1 to us-west-2; error window measured by k6 |

## Where the decisions are explained

See the [ADR index](../adr/README.md).
