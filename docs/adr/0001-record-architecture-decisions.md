# ADR-0001: Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

This project is a learning and showcase reference. A reader needs to
understand not only *what* was built but *why* — and which alternatives were
rejected. Decisions made in conversation are lost unless written down.

## Decision

Record every significant design decision as an ADR in `docs/adr/`, using the
template below. Decisions are immutable; a change of mind is a new ADR that
supersedes the old one.

```markdown
# ADR-NNNN: Title
- Status / Date
## Context      — the forces at play
## Options      — what we considered, with trade-offs
## Decision     — what we chose
## Consequences — what becomes easier, harder, or must be watched
```

## Consequences

- Reviewers of a pull request can check a change against the ADR it implements.
- Some overhead per decision; kept small by making ADRs short.
