# ADR-0014: Delegate `ha.noveon.io` from Namecheap to Route 53

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

`noveon.io` is registered at Namecheap and its DNS is served by Namecheap
hosting (`dns1/dns2.namecheaphosting.com`), which likely also serves the
existing website and email. The demo needs Route 53 for health-checked
failover records and for ACM certificate DNS validation.

## Options

| Option | Effect |
|---|---|
| Move all of `noveon.io` DNS to Route 53 | Must recreate every existing record (web, mail); risk of breaking email |
| **Delegate only `ha.noveon.io`** | Add 4 `NS` records at Namecheap; Route 53 is authoritative for `*.ha.noveon.io` only |

## Decision

Create a public hosted zone `ha.noveon.io` in the workload account
(permanent layer) and add its four name servers as `NS` records for host `ha`
in the Namecheap DNS zone. The existing site and email are untouched.

## Consequences

- One manual step at Namecheap (documented in runbook 02), done once.
- ACM certificates for `ha.noveon.io` / `*.ha.noveon.io` are requested in
  both regions and validated automatically through the delegated zone.
- If the hosted zone is ever deleted and recreated, it gets **new** name
  servers and the Namecheap records must be updated — one reason the zone is
  in the permanent layer.
