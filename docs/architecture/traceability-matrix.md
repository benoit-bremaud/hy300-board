# Traceability matrix

One row per use case and per non-functional requirement no use case realises. Sources are the brick issues (`#n → REQ`). Tests are the runbook pass/fail lists and the milestone acceptance checks until automated tests exist; `SEQ-movie-night` is the ID of `diagrams/projector-integration/02-sequence-movie-night.md`.

| UC or NFR | Source | Requirements | Realisation(s) | Components | Classes | States | Data flow | Tests | Status |
|---|---|---|---|---|---|---|---|---|---|
| UC1 Start a movie night | #10 → REQ-1, #11 → REQ-2, #11 → REQ-3 | REQ-1, REQ-2, REQ-3 | REQ-1 → SEQ-movie-night, REQ-2 → SEQ-movie-night, REQ-3 → SEQ-movie-night | HA scene, IR blaster, box, projector, optional plug sensor | n/a — no domain model | Projector: Off → Booting → VendorLauncher → HdmiInput | n/a — no personal data | REQ-1 → T-1 (three cold cycles), REQ-2 → T-4 (entities ×3), REQ-3 → T-2 (one-tap check) | planned v0.2 |
| UC2 Pick and play a film | #12 → REQ-4, #13 → REQ-4 | REQ-4 | REQ-4 → `03-component.md` (box web player node) + v0.3 player MCDA | box web player, Caddy, PopCornTV | n/a | n/a | n/a | REQ-4 → T-3 (≤ 3 presses from the PopCornTV home, login persists) | planned v0.3 |
| UC3 Power the setup off cleanly | #10 → REQ-1, #11 → REQ-2 | REQ-1, REQ-2 | REQ-1 → `03-component.md` text (trivial), REQ-2 → `03-component.md` text (trivial) | HA, box integration, IR blaster | n/a | Projector: HdmiInput → Off | n/a | REQ-1 → T-1, REQ-2 → T-4 | planned v0.2 |
| UC4 Identify the board | #2 → REQ-5 | REQ-5 | REQ-5 → `runbooks/01-identification.md` | projector, PC | n/a | n/a | redaction (`security-redaction.md`) | REQ-5 → T-5 (runbook 01 pass/fail list) | planned v0.1 |
| UC5 Back up the firmware | #3 → REQ-6, #3 → NFR-6 | REQ-6, NFR-6 | REQ-6 → `runbooks/02-dump.md`, NFR-6 → `runbooks/02-dump.md` | projector, PC | n/a | n/a | dump custody | REQ-6 → T-6 (identical hashes, or identical read-only partitions on a live read; GPT; `super`), NFR-6 → T-14 (grep: no `wl`, `ef`, `ul`, `fastboot flash` in v0.1 runbooks) | planned v0.1 |
| UC6 Capture IR codes and power facts | #4 → REQ-7 | REQ-7 | REQ-7 → `runbooks/04-ir-capture.md` | projector, remote, IR receiver, smart plug | n/a | Projector (all `?` guards) | n/a | REQ-7 → T-7 (runbook 04 table) | planned v0.1 |
| UC7 Reproduce the setup | #14 → REQ-8, #14 → NFR-7 | REQ-8, NFR-7 | REQ-8 → `runbooks/` 00–05 + v0.2/v0.3 runbooks (planned), NFR-7 → `runbooks/` | all | n/a | n/a | n/a | REQ-8 → T-8 (dry run, zero blocking deviations), NFR-7 → T-8 | planned v1.0 |
| NFR-1 Homelab security | #9 → NFR-1 | NFR-1 | NFR-1 → `08-deployment-home.md`, NFR-1 → ADR-0004 | guest SSID, Tailscale ACL, Caddy | n/a | n/a | n/a | NFR-1 → T-9 (ACL read back; 403 from the box on every host but PopCornTV; hotspot isolation pass in runbook 03) | planned v0.2 |
| NFR-2 Redaction | #7 → NFR-2 | NFR-2 | NFR-2 → `security-redaction.md` | `tools/redact.sh`, pre-commit | n/a | n/a | capture → redact → issue or commit | NFR-2 → T-10 (redact.sh fixtures, gitleaks, EXIF check) | planned v0.1 |
| NFR-3 Latency | #11 → NFR-3 | NFR-3 | NFR-3 → SEQ-movie-night | HA, box | n/a | n/a | n/a | NFR-3 → T-11 (≤ 60 s ×3) | planned v0.2 |
| NFR-4 Playback | #12 → NFR-4 | NFR-4 | NFR-4 → v0.3 player MCDA | box, Pi | n/a | n/a | n/a | NFR-4 → T-12 (2 h, < 0.5 % dropped, no throttling) | planned v0.3 |
| NFR-5 Safety | #2 → NFR-5 | NFR-5 | NFR-5 → `runbooks/00-safety.md` | maintainer | n/a | n/a | n/a | NFR-5 → T-13 (checklist ticked in the issue) | planned v0.1 |
