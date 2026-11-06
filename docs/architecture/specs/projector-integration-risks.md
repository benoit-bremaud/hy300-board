# Projector integration — risk register

Risks that can change a decision or block a milestone, with the fact that retires each one.

| ID | Risk | Impact | Retired by | Fallback |
|---|---|---|---|---|
| R1 | IR power-on from full off does not work (or the remote is Bluetooth-only). | REQ-1, ADR-0005 | runbook 04 | smart plug + auto-boot if R2 is clear; else manual power-on |
| R2 | The unit does not auto-boot on mains return. | ADR-0005 fallback | runbook 04 | manual power-on |
| R3 | The firmware boots to the vendor launcher and forgets the HDMI input. | REQ-3 (extra step) | runbook 04 | IR input selection in the scene |
| R4 | No box passes both gates (Tailscale, CA trust). | v0.2 | box MCDA | bb-homelab #28 (public domain + ACME) first |
| R5 | The USB-A port is not OTG: no USB dump path. | REQ-6 | runbook 01 | network ADB over the PC hotspot with the capture running; else recorded no-go |
| R6 | The dump holds per-unit keys; a leak exposes the unit. | NFR-2 | runbook 02 custody | hashes only, encrypted copies |
| R7 | Firmware-level malware contacts the Internet during recon. | NFR-1 | runbook 03 on an isolated hotspot | capture is the only contact; never the home SSID |
| R8 | Throughput over the tailnet (DERP relay) too low for direct play. | NFR-4 | ADR-0004 throughput gate | direct path check, box placement |
| R9 | HLS transcode on the Pi 5 (no hardware encoder) overloads it. | NFR-4 | v0.3 measurement | codecs chosen for direct play |
| R10 | Pre-existing: SSH `ALLOW Anywhere` on the LAN (bb-homelab AUDIT R13). | NFR-1 | out of scope, bb-homelab #232 | guest SSID isolates the box anyway |
| R11 | The projector firmware auto-joins an open or known Wi-Fi (operator community hotspots) or pairs BT on its own. | NFR-1 | runbook 01 step 1 (Wi-Fi and BT off, no saved network) | never enter any home PSK on the projector |
| R12 | The IR blaster (a cloud-bound Broadlink, for instance) sits on the main LAN. | NFR-1 | v0.2 blaster choice: ESPHome local-only preferred | Broadlink in local mode, cloud blocked at the router |
| R13 | The "isolated" capture hotspot NATs the projector onto the LAN or the tailnet. | NFR-1, R7 | runbook 03 isolation pass/fail | separate WAN, `tailscale down`, drop rules |
