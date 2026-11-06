# Runbook 04 — IR codes and power facts (UC6, REQ-7, ADR-0005)

Precondition: runbook 00; runbook 02 done or its no-go recorded (the auto-boot test hard-cuts mains: no cut before a backup attempt). Needs an IR receiver to capture and a blaster (or the same ESP) to replay. No network needed. Under strategy C run it **before** runbook 03, unless the OTG branch of runbook 01 already ran it.

## IR codes

1. Confirm the remote is infrared: cover the projector's IR window with opaque tape, or watch the remote's emitter through a phone camera (IR shows as a flash). If the projector still reacts with the window covered, it is Bluetooth/RF (record R1).
2. Capture with an IR receiver (ESPHome `remote_receiver` with `dump: all`, Broadlink learn mode, or a Flipper/Arduino sketch): `POWER`, `INPUT`/`SOURCE`, `OK`, arrows, `BACK`, `HOME`, `VOL+`, `VOL-`, `MUTE`.
3. Record protocol (NEC, …), address, command per key in `docs/research/<date>-ir-codes.md`.

## Power and input facts

| Check | How | Result |
|---|---|---|
| Codes captured per key, replay reproduces the key | replay each code | |
| Standby draw (W) after IR POWER off | smart plug with power reading | |
| IR POWER turns the projector on from "off" (mains kept) | replay the code | |
| Auto-boot on mains return | unplug 1 min, replug | |
| Boots to the HDMI input, or remembers the last input | select HDMI, power cycle | |
| IR INPUT (or SOURCE + arrows + OK) selects HDMI from the vendor launcher | replay the sequence | |
| Boot time to picture (s) | stopwatch | |

## Outcome

Freeze ADR-0005: option **ir** (IR on and off) stays if IR POWER from off passes; fallback **plug** (smart plug + auto-boot, IR off) if it fails and auto-boot passes; else **manual**. If IR power-on and auto-boot both fail, or IR input selection fails while the input is not remembered, apply ADR-0003's review clause (runbooks 03 then 05).
