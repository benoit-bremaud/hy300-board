# Roadmap — projector-integration

The living order of the bricks of epic #1, one GitHub issue each, with what each needs and what "done" means. Plan §5 of `docs/research/2026-11-06-framing-and-plan.md` is the frozen record of the milestones; this file is updated at every merged brick (one chat per brick, `PROJECT_LOG.md` keeps the history).

Last updated: 2026-11-06, after PR #16 (brick #2).

## Where we are

| Milestone | Bricks | State |
|---|---|---|
| v0.1 — Reconnaissance | #2 ✅ (PR #16), #17, #18, #3, #4, #7, (#5, #6 conditional) | #2 done; the rest waits on the bench or runs hardware-free (below) |
| v0.2 — Home Assistant | #8, #9, #10, #11 + bb-homelab #234, #235, #236 | not started; #8 is hardware-free |
| v0.3 — PopCornTV on screen | #12, #13 | not started |
| v1.0 — Reproducible release | #14, #15 | not started |

Decisions still open: the ADR-0002 matrix weights (40/30/30) are `proposed` and are put to the owner at the start of the next brick; ADR-0003 (strategy C) is confirmed or re-scored after #4 (review clause); ADR-0005 (`ir`) is frozen by #4's results.

## Order

The two tracks run in parallel: hardware-free bricks whenever the owner is away from the unit, bench bricks in the mandatory order of plan §5 (no network before the dump; under C, #4 before #5).

### Track 1 — hardware-free (next up)

| # | Brick | Depends on | Done when |
|---|---|---|---|
| #17 | chore(ci): wire the local CI tiers (just, lefthook, mise), docs-only profile | — | `mise x -- just check` green from a clean environment; hooks refuse a staged secret |
| #7 | feat(tools): redaction script with tests and a pre-commit check | #17 (the check hooks into lefthook) | `tools/redact.sh` covered by tests (MAC, serial, SSID, IP, IPv6 patterns of `security-redaction.md`); runbooks 01–03 and 05 pipe through it |
| #8 | docs(ha): choose the HDMI box by MCDA with pass/fail gates | ADR-0003 confirmed, or accepted as provisional | ADR-0006 with gates (runs Tailscale; trusts the Caddy root in the player used, or bb-homelab #28 first; HA integration ports listed for the ACL) |
| bb#234 | feat(tailscale): default-deny ACL for `tag:media-box` | — (bb-homelab chat) | ACL merged, tag and pre-auth key documented |
| bb#235 | feat(caddy): 403 to the media box on every host but PopCornTV | bb#234 (the box's tailnet IP range) | matcher merged with a test |
| bb#236 | feat(dns): resolve bb-homelab names for tailnet clients | — | split DNS on the Pi, or #28 delivered |

### Track 2 — on the bench (plan §5 order, runbook 00 first every session)

| # | Brick | Depends on | Done when |
|---|---|---|---|
| #18 | feat(recon): identify the board on the bench (runbook 01) | runbook 00 read; Wi-Fi and Bluetooth off before anything | REQ-5 facts recorded and redacted; OTG branch decided |
| #3 | feat(recon): back up the firmware read-only and verify the dump (runbook 02) | #18 (OTG port known); #7 for the paste of any output | T-6 pass per path, or the no-go recorded; T-14 clean |
| #4 | feat(recon): capture the IR codes and the projector power facts (runbook 04) | #3 (nothing powers up on the bench before the dump) | REQ-7 and REQ-8 rows filled; ADR-0005 frozen; ADR-0003 review clause evaluated |
| #5 | feat(recon): egress capture on an isolated hotspot (runbook 03) | only if #4 lowers C (ADR-0003 review clause) | destination list reviewed; isolation pass/fail recorded |
| — | cold-boot checks (runbook 05) | with #5, same hotspot session | ADB 5555 after cold boot recorded (A's cell) |
| #6 | chore(recon): UART boot log (optional) | #7 (log through the redaction script) | boot log in `uart-logs/`, SoC confirmed |

v0.1 acceptance (plan §5): SoC and build identified; dump verified or the no-go recorded; pass/fail recorded for OTG, each dump path, standby, auto-boot, IR codes, IR power-on, IR input selection, HDMI input memory; ADR-0003 confirmed; ADR-0005 frozen.

### v0.2 → v1.0

1. #9 feat(network): box on the guest SSID and the tailnet — after bb#234, bb#235, bb#236 and #8 (the box exists). Done: the D3 checklist of plan §5 step 8 run on the box; throughput gate measured.
2. #10 feat(ha): projector power and HDMI input through the IR blaster — after #4 (codes) and #9. Done: HA drives power and input at least once; the plug-reading retry exists only if a plug is present (UC1 2a).
3. #11 feat(ha): box `media_player` and the movie-night scene — after #9, #10. Done: v0.2 acceptance (three cold boots in a row, NFR-3 ≤ 60 s measured).
4. #12 docs(player): measure and choose the player by MCDA — after #11. #13 feat(player): dedicated account and persistent session — after #12. Done: v0.3 acceptance (2 h 1080p, < 0.5 % dropped frames, login survives a reboot, remote-only start).
5. #14 docs(release): reproduce-at-home guide and third-party dry run — after v0.3. #15 chore(release): tag v1.0.0 and the QA pass — after #14. Done: v1.0 acceptance (≤ 3 presses + 1 tap from a cold projector; traceability matrix complete).

## Rules for every brick

- One chat named `#N <issue title>`, branch `<type>/<N>-<slug>` from `main`, PR closing the issue, pre-push adversarial gate open, `review-local.md` updated, `PROJECT_LOG.md` entry, then this file.
- A bench brick starts with runbook 00 and ends with its facts in `docs/research/` and the traceability matrix row set to `done` or `no-go`.
- A fact that moves a cell of ADR-0003, ADR-0004 or ADR-0005 re-scores the matrix in the same PR (review clauses).
