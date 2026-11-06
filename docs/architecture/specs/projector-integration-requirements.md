# Projector integration — requirements

Functional (`REQ-n`) and non-functional (`NFR-n`) requirements of epic #1, each with a measurable acceptance criterion. Source: `docs/research/2026-11-06-framing-and-plan.md` §5 acceptance lists.

## Functional

| ID | Requirement | Acceptance | Milestone |
|---|---|---|---|
| REQ-1 | HA powers the projector on and off and selects the HDMI input through the IR blaster. | From a cold projector, an HA service call lights the panel on the HDMI input; a second call powers it off; three times in a row. | v0.2 |
| REQ-2 | HA drives the box: state, power, volume, app launch, remote keys, through its integration over the tailnet. | Every entity works from a cold boot three times in a row; host set by hand. | v0.2 |
| REQ-3 | A "movie night" scene started by one user tap turns everything on and lands on PopCornTV. | Lights, projector on + HDMI, box on, PopCornTV launched, notification; one tap. | v0.2 |
| REQ-4 | A family member picks and plays a film with the remote only. | ≤ 3 remote presses from the PopCornTV home to playback (the single definition, reused by plan v1.0); login persisted across reboots. | v0.3 |
| REQ-5 | The board is identified without any network contact. | About screen (redacted), `getprop` values, OTG port recorded; Android version and SoC vendor known. | v0.1 |
| REQ-6 | A read-only dump of the eMMC is obtained and verified, or the no-go is recorded. | Two reads with identical SHA-256 (whole image with the OS not running; per read-only partition on a live read), GPT parsed, `super` unpacked; hashes in `dumps/README.md`; or each path's failure recorded. | v0.1 |
| REQ-7 | The remote's IR codes and the projector's power facts are captured. | Codes captured and replayed per key; pass/fail for: remote is IR, power-on from full off, auto-boot on mains return, HDMI input remembered, IR input selection from the vendor launcher, standby draw in W, boot time. | v0.1 |
| REQ-8 | A third party reproduces the setup from the repository alone. | Dry run by a non-author (or the owner on a fresh machine and account): zero blocking deviations. | v1.0 |

## Non-functional

| ID | Requirement | Measure | Realised by |
|---|---|---|---|
| NFR-1 | Homelab security: the projector never joins the home network or the tailnet; its only network contact under C is the isolated hotspot of runbooks 03 and 05; the box is isolated (guest SSID, default-deny ACL, Caddy 403). | ACL read back in the admin console; from the box, every host but PopCornTV returns 403; the projector's Wi-Fi is never configured with a home SSID. | ADR-0003, ADR-0004, `08-deployment-home.md` |
| NFR-2 | Redaction: no serial number, MAC, SSID/PSK, Tailscale key or IP in any committed artefact. | `tools/redact.sh` on every capture, covered by a test; pre-commit grep on MAC and serial patterns. | `security-redaction.md` |
| NFR-3 | Latency: ≤ 60 s from power-on to HA `on` for the box. | Measured three times. | `02-sequence-movie-night.md` |
| NFR-4 | Playback: 1080p direct play for 2 h with < 0.5 % dropped frames and no thermal throttling on the box; Pi CPU load recorded. | Player measurement of v0.3. | v0.3 MCDA |
| NFR-5 | Safety: the safety preamble is applied before any hardware step. | Runbook 00 checklist signed in the issue. | `runbooks/00-safety.md` |
| NFR-6 | No write to the projector's eMMC in v0.1; any later write needs its own ADR and a proven loader. | No `rkdeveloptool wl`, `ef`, `ul` or `fastboot flash` in any runbook of v0.1 (T-14). | `runbooks/02-dump.md` |
| NFR-7 | Reproducibility: every procedure is a runbook with its pass/fail list and the list of what a third party brings. | REQ-8 dry run. | `runbooks/` 00–05 now; v0.2 (box, network, IR blaster) and v0.3 (player) runbooks are written with their bricks |
