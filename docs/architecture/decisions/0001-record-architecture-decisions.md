# ADR-0001 — Record architecture decisions

Status: accepted · Date: 2026-11-06 · Last changed: pre-push review round 2

## Context

The project turns an untrusted consumer projector into a homelab device; its decisions (strategy, network, power) were reached through weighted matrices and several review rounds. Without a record, a third party cannot tell why the projector is kept off the network or why the box exists.

## Decision

Record every significant decision as a MADR file under `docs/architecture/decisions/`, numbered `NNNN` monotonically, titled as a verb phrase, with status in the file. Each ADR that chooses between scored options cites its MCDA matrix (weights /100, scores 0–5, totals /500, sensitivity) with the cell reasons, and carries a `Last changed:` line naming the plan revision and review round. Sub-decision options are named with words (`ir`, `plug`, `guest-ts`), never the letters A/B/C reserved for the strategies.

## Consequences

- Decisions are traceable from the matrix (`traceability-matrix.md`) and from the plan.
- A superseded ADR keeps its number and points to its successor.
