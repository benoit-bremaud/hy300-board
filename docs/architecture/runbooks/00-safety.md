# Runbook 00 — safety preamble (NFR-5)

Read and apply before every hardware runbook; copy the checklist into the issue and tick it. Output pasted into an issue goes through `tools/redact.sh` first (`specs/security-redaction.md`).

## Mains and stored energy

- [ ] Mains unplugged; wait 2 min. The power board is an **off-line mains supply** (`AC100-240V` on its label): the dangerous charge sits on the **primary bulk capacitor (~325 V DC)**, not on the 27 V LED rail. Measure across the bulk capacitor's terminals; if above 50 V, discharge it through a resistor (≥ 1 kΩ, 5 W) and measure again.
- [ ] Then measure the ~27 V LED rail and the 5 V rail; both near 0 V before touching the boards.
- [ ] Before any powered step with the case open, cover or insulate the supply board's primary side; preferred: power the mainboard from a bench 5 V supply on `5V_IN` (`5V 5V GND GND EN`) for runbooks 01, 02 and the UART step, with the mains supply unplugged.

## Board and optics

- [ ] ESD strap on; work on a non-conductive surface.
- [ ] Never power the board with the heatsink lifted or the fan unplugged (SoC thermal runaway); re-pad the heatsink if it was lifted.
- [ ] Optics open → eye protection; never look into the LED module when powered.

## Cables and probes

- [ ] USB A-to-A cable with VBUS cut (or a data-only adapter) between PC and projector; never back-feed 5 V.
- [ ] UART adapter set to 3.3 V, level checked with a meter on `TX` before wiring; GND first.
- [ ] Maskrom short (runbook 02 path 3) only with the mains supply unplugged and the bench 5 V supply off, then powered on with the short held; photo of the short point before doing it.

## Network

- [ ] Wi-Fi and Bluetooth off on the projector as the first action after boot (runbook 01); any network contact only on the isolated hotspot of runbook 03, never the home SSID.
