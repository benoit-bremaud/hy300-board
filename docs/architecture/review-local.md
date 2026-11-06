# Local review — hy300-board (`--push`)

- Date: 2026-11-06
- Commit: 0f37a19 (the squashed branch, checked at `ccda442` before the squash); head 230e438
- Scope: branch diff `main...docs/2-board-recon` (4 commits: the squashed dossier, then the log, lint and report commits; round SHAs below refer to the pre-squash local history, tag `local/board-recon-history-2026-11-06`, not pushed)
- Checks run: adversarial (Opus, 3 rounds) · docs (doc-auditor, round 1) · secrets · Mermaid render · design, tests, correctness: n/a (documentation-only branch, no code)
- Verdict: pushed, PR #16 open · Must 0 · Should 0 · Nice 0 (all applied)

## Adversarial review (#236)

- Round 1 on `43949d8`: Must 10 (13 reconciled with the doc audit) · Should 19 · Nice 9 · unverified 0 · Push gate: closed
- Round 2 (verify `916df42`): 9 fixed · 1 not fixed (stale copy in the plan) · 1 regressed (ADR-0002 sensitivity) · new Must 3 · Push gate: closed — 4 Must open
- Round 3 (verify `ccda442`): 4 Must fixed · 7 Should fixed · 6 Nice fixed · new Must 0 · new Should 1 (photo history) · new Nice 3 · Push gate: open
- Round 4 (verify `230e438`, rebuilt branch): photo blobs in `main..HEAD` verified EXIF-free, Should and 3 Nice fixed · new Must 0 · new Should 1 (this report, fixed here) · **Push gate: open**
- Hosted reviewers (filled at the pre-merge run): Copilot pending · Codex optional

## Pre-merge gate (head SHA 230e438)

| Step | Result |
|---|---|
| 0 CI mirror | n/a — no `.github/workflows/` on this branch (filed as a v1.0 brick) |
| 1 Tests | n/a — no code, no test suite |
| 2 Coverage | tests n/a · docs ⏸ deferred to the pre-merge run (`--push`) |
| 3 SonarQube | ⏸ deferred to the pre-merge run (`--push`); no SonarQube project yet |
| 4 Secrets | ✅ gitleaks clean (`main..HEAD` and full history by the reviewer) · no tracked .env · diff read: no MAC, IP, SSID, key or serial; EXIF stripped from the 9 committed photos at ccda442 |
| 5 E2E/smoke | ⏸ deferred to the pre-merge run (`--push`); no runnable product |

- Gate: `--push`: **pre-push green** — steps 1, 2 and 4 by hand (no `justfile`), steps 2 (docs), 3 and 5 deferred (never GREEN).
- Local CI, docs-only profile of skill `local-ci`, run by hand after the push (no `justfile`, `mise.toml` or hooks yet): `typos .` clean (`typos.toml` allows Mosquitto) · `markdownlint-cli2 "**/*.md"` 0 errors on 29 files (`.markdownlint-cli2.jsonc`: MD013 and MD060 off) after 24 real findings were fixed (bare URLs, heading and list spacing, one fence language) · `lychee --offline` 0 errors · `gitleaks git --log-opts=--all` no leaks. Wiring the tiers (`just check`, lefthook, mise pins) is a separate brick.
- Mermaid: the 6 diagram blocks render with `mmdc` 11.12.0, no syntax error.

## 🔴 Must Have

_None._

## 🟡 Should Have

_None open._ Fixed before the first push: the original JPEGs with phone make/model EXIF existed only in the local history; the branch was rebuilt from `main` as one commit (`0f37a19`, owner's choice `squash-one`), and round 4 verified that the 9 blobs in `main..HEAD` carry no EXIF.

## 🔵 Nice to Have

_None open._ Applied in the report commit: dedicated `nft` table `inet hy300-capture` with `delete table` cleanup (runbook 03), "5G operator router" in plan §2, round-1 Must count detailed in `PROJECT_LOG.md`.

## ⚪ Disagree / Intentional

- `docs/research/2026-11-06-framing-and-plan.md` §8 `[adversarial]` · the changelog keeps the historical A/B/C letters of earlier rounds · it is a log; the live rows of §4 use the word names (ADR-0001).
- `docs/architecture/runbooks/` `[docs]` · layout deviation from `conception-naming` · stated in `README.md` § How to read; NFR-7 is realised by the runbooks.

## Per-dimension summary

- Adversarial: 3 rounds, 10 + 3 Must fixed and verified, gate open; one Should on the photo history left to the owner. · Design: n/a. · Docs: round-1 doc audit merged into the Must list (3 Must), all fixed. · Tests: n/a. · Security: gitleaks clean, EXIF stripped in the tree, no private value in any diagram or runbook. · Correctness: ADR arithmetic (0002, 0003, 0004, 0005) recomputed by the reviewer and matching.
