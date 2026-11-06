# ADR-0002 — Adopt the hybrid UP + ICONIX conception method

Status: accepted for the method (owner choice D8, validated 2026-11-06); the matrix weights below were set in pre-push review round 1 and are proposed, pending the owner's validation · Date: 2026-11-06 · Last changed: pre-push review round 2

## Context

The owner's rule is UML-first with a full dossier. The project mixes an unknown risk (unidentified SoC, untrusted firmware, four physical devices) with a small, behaviour-centred feature set (one scene, a handful of entities).

## Decision drivers

- Risk must be retired before building (hardware unknowns decide the plan).
- The feature set is small: a full UP artefact set would be ceremony.

## Options considered (fit to the risk 40, cost of ceremony 30, speed to a reviewable design 30)

| Option | Fit | Ceremony | Speed | Total /500 |
|---|---|---|---|---|
| UP alone | 5 : risk-driven iterations | 2 : full artefact set for four devices | 2 : every artefact before a reviewable design | 200+60+60 = **320** |
| ICONIX alone | 2 : no risk phasing | 5 : minimal subset, four review gates | 5 : shortest path from use case text to design | 80+150+150 = **380** |
| **Hybrid** (UP phases, ICONIX path per use case — the owner's default method for a full dossier) | 5 : UP phases retire the hardware risk first | 4 : only the views that answer a question | 4 : ICONIX path per use case, one phase of framing on top | 200+120+120 = **440** |

Sensitivity (others renormalised proportionally): ICONIX alone passes hybrid when fit weighs below 25 (at 24: ICONIX 428 vs hybrid 424; at 25 they tie at 425). D8 itself (full dossier vs spec-only) was an owner choice: the project touches four devices and carries unknown hardware risk, the full-dossier threshold of the method.

## Outcome

Hybrid. Inception = framing session; Elaboration = v0.1 reconnaissance; Construction = v0.2–v0.3; Transition = v1.0. Per use case: Cockburn text → sequence → components. Diagrams kept: use case, sequence, component, state, deployment. Skipped: class (no domain model), data-flow (covered by `specs/security-redaction.md`), activity (runbooks are the procedures).

## Consequences

- Each milestone revisits the matrix and the ADRs with the facts it produced (ADR-0003 carries a review clause and ADR-0005 a "frozen after runbook 04" clause).
