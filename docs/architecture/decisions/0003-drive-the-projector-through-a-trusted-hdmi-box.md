# ADR-0003 — Drive the projector through a trusted HDMI box

Status: accepted (re-scored only if a v0.1 fact moves a cell, see § Review clause) · Date: 2026-11-06 · Decision D1, weights validated by the owner on 2026-11-06 · Last changed: plan revision 6, pre-push review round 1

## Context

The projector's firmware is untrusted (malware history in the product family, firmware-level implants); its SoC is unidentified (Allwinner H713 ruled out, Rockchip likely). HDMI is an input: a trusted external device can feed the panel. Home Assistant and PopCornTV live on the homelab Pi behind Caddy and a tailnet.

## Decision drivers (weights /100)

Reaches the goal 30 · homelab security 20 · risk/reversibility 20 · learning value 15 · cost 10 · playback quality 5.

## Options considered

| Option | Goal | Security | Risk | Learning | Cost | Playback | Total /500 |
|---|---|---|---|---|---|---|---|
| A. Stock Android + ADB from HA | 4 : HA via ADB, PopCornTV in a browser on a weak 2 GB box | 2 : untrusted firmware on the tailnet behind an ACL to build | 4 : no flashing, ADB changes undone | 4 : ADB, dump, UART, HA | 3 : 0 € but ACL, DNS, CA, ADB persistence | 3 : stock browser | 345 |
| B. Linux on the board | 3 : unknown SoC, port may not exist | 4 : own OS, vendor blobs remain | 1 : flash on an unidentified SoC | 5 : the whole stack | 1 : weeks, uncertain | 2 : decode not guaranteed | 285 |
| **C. Trusted HDMI box, projector off-network** | 4 : PopCornTV on a box HA drives; projector only gets IR | 5 : projector never on a network | 5 : nothing on the projector changes | 1 : projector stays a black box | 2 : box 40–80 € + blaster | 5 : current streaming box | **380** |

Sensitivity (others renormalised proportionally): A passes C when learning rises above ≈ 25.6 at security 20 (26/20: A 347.5 vs C 346.2), or above ≈ 21.5 at security 15 (22/15: A 355.5 vs C 353.8).

## Outcome

C. A trusted box plays PopCornTV and is driven by HA; the projector is a display plus IR for power and input, and never joins the home network or the tailnet after the recon (its only contact: the isolated hotspot of runbooks 03 and 05). A is the documented alternative; B is a study (v3) opened once the SoC and a loader are known.

## Review clause

v0.1 facts that can move a cell: IR power-on from full off and auto-boot both failing, or IR input selection failing while the firmware forgets the HDMI input, lowers C to 350; a clean egress capture plus ADB 5555 surviving a cold boot raises A to 375. If runbook 04 lowers C, runbook 03 (egress capture) and runbook 05 (cold-boot checks) run on the same isolated hotspot before this ADR is confirmed; only both movements together flip it.

## Consequences

- v0.2 adds a box choice (MCDA with pass/fail gates: runs Tailscale; trusts the Caddy root in the player used, or bb-homelab #28 delivered first) and an IR blaster.
- The projector gets no debloat, no launcher, no network: D5 and the network decision apply to the box.
- Learning on the board itself is limited to the read-only recon (dump, UART) kept as the v3 backup.
