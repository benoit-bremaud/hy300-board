# ADR-0005 — Control the projector power by IR (power on/off and HDMI input)

Status: accepted, frozen after the v0.1 power tests · Date: 2026-11-06 · Decision D4, validated by the owner on 2026-11-06 · Last changed: plan revision 6, pre-push review round 1

## Context

Under ADR-0003 the projector has no network, so HA cannot wake it over ADB. An IR blaster is required anyway for power-off and HDMI input selection. The `POWER` button is momentary; whether the unit auto-boots on mains return and whether IR power-on works from full off are unknown until v0.1.

## Decision drivers (weights /100)

Works anywhere 30 · family UX 25 · hardware risk 20 · cost 15 · depends on an unverified fact 10.

## Options considered (IR blaster as a given)

| Option | Anywhere | UX | HW risk | Cost | Unverified | Total /500 |
|---|---|---|---|---|---|---|
| plug: smart plug + auto-boot, IR off | 3 : same blaster plus a plug and Wi-Fi for it | 4 : one scene | 2 : hard cut at every off | 4 : plug owned | 1 : auto-boot untested | 300 |
| manual: manual on, IR off, HA follows | 3 : the blaster travels with the box | 2 : power on first, then select the input | 5 : nothing forced | 3 : blaster anyway | 2 : state after a cold boot untested | 305 |
| **ir: IR on and off, IR input selection** | 3 : same kit | 4 : one scene | 5 : the remote's own commands | 3 : blaster anyway | 3 : IR power-on from full off untested | **365** |

Sensitivity (others renormalised proportionally): `ir` stays on top at every hardware-risk weight; `plug` passes it only if cost weighs ≈ 49 or more (at 50: 341 vs 338).

## Outcome

`ir`. HA drives the projector's power and HDMI input through the IR blaster with the codes captured in runbook 04; volume is set on the box's `media_player` (audio travels over HDMI). Fallback if IR power-on from full off fails and auto-boot is confirmed: `plug` (smart plug for power-on, IR for power-off); else `manual`.

## Consequences

- Runbook 04 records pass/fail for: codes captured and replayed per key, remote is IR (not Bluetooth-only), IR power-on from full off, auto-boot on mains return, HDMI input remembered, IR input selection from the vendor launcher, standby draw, boot time.
- The "movie night" scene is one HA tap: projector on, HDMI selected, box on, PopCornTV launched.
