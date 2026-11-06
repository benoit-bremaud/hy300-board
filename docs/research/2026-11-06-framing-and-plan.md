# Framing session and action plan — 2026-11-06 (revision 7, owner-validated weights)

Outcome of the framing Q&A held on 2026-11-06, before any hardware work, revised after the author's critical reread, seven adversarial review rounds and two pre-push review rounds (section 8 is the changelog). Status: **weights, D2, D3 (box variant `guest-ts`), D4–D8 validated by the owner on 2026-11-06**; the strategy follows the top score; this document seeds `docs/architecture/`.

## 1. Need

Turn the HY300-branded projector into a device that Home Assistant (bb-homelab, Pi 5) controls and that shows PopCornTV, for family movie nights (owner and two daughters, guests), usable by everyone: remote control, phone, HA dashboard, PC. The projector is nomadic (room to room now, other places later). Everything must be reproducible by a third party from the repository plus a stated list of what they bring (section 6), to the extent the v1.0 dry run covers.

## 2. Facts that constrain the design

| Fact | Source | Consequence |
|---|---|---|
| **SoC vendor unknown; Allwinner H713 ruled out.** The board carries a Rockchip RK621 HDMI→MIPI bridge (the H713 has HDMI input built in and would not need one), a `RST/RECOVERY` button, a Samsung eMCP 2 GB + 16 GB and a 24 MHz crystal; the SoC sits under the heatsink. Rockchip is the likely vendor, not a proven one. | teardown photos, `2026-11-06-board-identification.md` | Vendor-specific steps (Loader/Maskrom, `rkdeveloptool`, 1 500 000-baud console) are conditional on the About screen or the USB vendor id (`2207` Rockchip, `1f3a` Allwinner, `1b8e` Amlogic). H713 material is background only. |
| **HDMI is an input.** The RK621 feeds the panel from the HDMI port; the projector cannot output to another screen. | RK621 datasheet, photos | A trusted external box can drive the panel (strategy C). The Android side is needed only if the projector itself must play (strategy A). |
| Firmware of this product family has shipped **malicious code** (BadBox-type implants live in firmware and native services, not only in APKs); the vendor statement covers H713 units, not this board. | Magcubic statement, press | Stock firmware is untrusted whatever the board: egress capture on first network contact, isolation, hash-only sharing of the dump. |
| Home Assistant runs on the Pi 5 in host network; Caddy and Jellyfin are bound to `127.0.0.1` + tailnet IP (ADR 0007); Caddy serves `*.bb-homelab.local` with an internal CA and clients use `/etc/hosts`; the tailnet ACL is **probably allow-all** (inferred from `tailscale.md` Pattern B; confirmed in v0.1 step 8 under A); SSH 22 and Mosquitto 1883 are open on the LAN; the router is a **5G operator router** (CGNAT, no port forwarding, VLAN unknown). | bb-homelab `network/`, `docs/security/AUDIT.md`, `services/caddy/Caddyfile` | From the LAN, PopCornTV is unreachable by design. Any client (projector under A, box under C) needs the tailnet, name resolution for `popcorntv.bb-homelab.local` and trust in the Caddy root, or the public domain of bb-homelab #28 with ACME. |
| PopCornTV requires HTTPS (secure cookies, `PUBLIC_URL`, origin check, ADR-0010); a user-installed CA is trusted by Chrome but **not by WebView apps** by default; Kodi has no PopCornTV client. | popcorntv ADRs | Player candidates are judged on CA trust and codec support, not only on playback. |
| A powered-off Android projector answers nothing; the `POWER` button is momentary; whether "off" is a Wi-Fi-alive standby is unknown. | photos | Remote power-on: IR (D4, to confirm from full off), smart plug + auto-boot (to test), standby + wake (A only), or manual. Whether the firmware remembers the HDMI input across power cycles is unknown. |
| `adb tcpip 5555` does not persist across reboots without root; Android 11+ "wireless debugging" uses pairing and a random port that the HA core integration does not use. | AOSP | Under A, network ADB surviving a cold boot is a recon pass/fail, not an assumption. |

## 3. Strategy matrix (weights validated by the owner, 2026-11-06)

The first matrix (PROJECT_LOG 2026-11-06, 375/345/245) had no security criterion and scored "external box" on a wrong fact. Re-run:

| Criterion | Weight | A. Stock Android + ADB | B. Linux on the board | C. Trusted HDMI box, projector off-network |
|---|---|---|---|---|
| Reaches the goal (HA + PopCornTV + family UX) | 30 | 4 : HA via ADB, PopCornTV in a browser/app on a weak 2 GB box | 3 : depends on an unknown SoC and a port that may not exist | 4 : PopCornTV on a box HA already knows how to drive; the projector itself only gets power/volume via IR or plug |
| Security of the homelab | 20 | 2 : untrusted firmware on the tailnet behind an ACL that must be built | 4 : own OS, vendor blobs remain | 5 : projector never on a network |
| Risk / reversibility | 20 | 4 : no flashing; ADB-only changes undone with `install-existing` | 1 : flash on an unidentified SoC with no proven loader | 5 : nothing on the projector changes |
| Learning value (hardware, UART, dumps, HA) | 15 | 4 : ADB, dump, UART, HA integration | 5 : U-Boot, device tree, loaders: the whole stack | 1 : a box to configure, the projector stays a black box |
| Cost (money and evenings) | 10 | 3 : 0 €, but ACL, DNS, CA and ADB-persistence work | 1 : weeks, outcome uncertain | 2 : box 40–80 € + IR blaster, few evenings |
| Playback quality | 5 | 3 : stock browser on a 2 GB SoC | 2 : hardware decode not guaranteed on a port | 5 : a current streaming box |
| **Total /500** | 100 | **345** | **285** | **380** |

Points: A = 120 + 40 + 80 + 60 + 30 + 15 = 345; B = 90 + 80 + 20 + 75 + 10 + 10 = 285; C = 120 + 100 + 100 + 15 + 20 + 25 = 380.

Sensitivity (others renormalised proportionally): at learning 25 / security 15, A 356 vs C 345; at learning 25 / security 20, C 349 vs A 347. The owner validated the proposed weights on 2026-11-06, so **C is the strategy**. What v0.1 can still change: if the egress capture is clean and ADB 5555 persists across a cold boot, A's "reaches the goal" could rise to 5 (A 375, still below C). What can lower C: IR power-on from full off fails (or the remote turns out to be Bluetooth-only) **and** auto-boot on mains return fails, or IR input selection fails while the firmware does not remember the HDMI input; then C's "reaches the goal" drops to 3 (C 350, still above A at 345, below A at 375). Only the combination of both movements flips the strategy. ADR 0003 records C now; A stays the documented alternative, re-scored only if a v0.1 fact moves a cell. B stays a study (v3).

## 4. Decisions

| ID | Decision | Rejected | Score |
|---|---|---|---|
| D1 | **Strategy C**: a trusted HDMI box plays PopCornTV and is driven by Home Assistant; the projector is a display plus IR for power and input selection and never joins the home network or the tailnet after v0.1 step 4. Levels in order: (1) HA drives the box, PopCornTV on screen → (2) playback driven from the phone → (3) Linux study on the board | A 345, B 285 | C 380 |
| D2 | Home network first; nomadic later | — (owner choice, validated) | — |
| D3 | *Strategy A only.* Network for the projector: box guest SSID + Tailscale (tagged node, pre-auth key, key expiry off) + tailnet ACL default-deny (projector → Pi `tcp:443` only, Pi → projector `tcp:5555` only) + Caddy 403 to the projector on every host but PopCornTV; gate: direct path and throughput ≥ 2× the highest direct-play bitrate. main-ts (main Wi-Fi + Tailscale) only by owner derogation recorded in a new ADR, since on the LAN the projector reaches SSH 22, Mosquitto 1883, the box UI and every device, and `adbd` 5555 is open to the LAN. Under C the **box** (trusted, but vendor firmware all the same) goes on the guest SSID + tailnet with its own tag, ACL and Caddy 403 (box matrix below, 430 vs 405 for main Wi-Fi); the projector has no network. | main-wifi-caddy 245, guest-only 340, main-ts 345 (derogation only) | guest-ts 460 |
| D4 | Power control under C: **IR blaster drives the projector's power and input selection** (the blaster is required anyway for power-off and HDMI selection); the box is powered by its HA integration; volume on the box's `media_player` (audio travels over HDMI). Recon captures the IR codes and tests power-on from full off, auto-boot on mains return, HDMI-input memory, standby draw; ADR 0005 frozen after those results. Replaces the session's "manual" choice (scored before C was adopted); **validated by the owner on 2026-11-06**. | plug 300, manual 305 | ir 365 |
| D5 | *Strategy A only.* Stock Android debloated over ADB (`pm uninstall -k --user 0 <pkg>`, undo `cmd package install-existing <pkg>`), TV launcher (Projectivy or Molla). Under C: the vendor UI is used only to select the HDMI input (by IR, or never if the firmware remembers the last source). | vendor UI, kiosk (owner choice, validated) | — |
| D6 | Player chosen by MCDA after measurement **on the box** (under A, on the projector): candidates Chrome/Chromium, full-screen WebView app (CA trust must be opted in), Kodi only if a client is written into scope. Pass/fail per candidate: CA trust, `canPlayType` for the library's codecs, 1080p direct play, remote navigation, persistent login. | choose now (owner choice, validated) | — |
| D7 | One brick = one issue = one chat; milestones v0.1 recon / v0.2 HA / v0.3 PopCornTV / v1.0; this framing is epic #1 work on branch `docs/2-board-recon` because it is the conception gate of #2 | larger batches (owner choice, validated) | — |
| D8 | Full conception dossier, hybrid UP + ICONIX (owner's UML-first rule) | spec-only (owner choice, validated) | — |

### D3 matrix (need 35, security 30, simplicity 20, bb-homelab coherence 15)

| Option | Need | Security | Simplicity | Coherence | Points | Total |
|---|---|---|---|---|---|---|
| main-wifi-caddy. Main Wi-Fi, Caddy reopened on LAN | 4 : works, opens the LAN | 1 : untrusted device sees everything | 3 : Caddy change + ADR 0007 reopened | 1 : breaks ADR 0007 | 140+30+60+15 | **245** |
| guest-ts. Guest SSID + Tailscale + ACL | 5 : PopCornTV, HA and phone all reachable | 5 : isolated by the box, ACL on the tailnet | 3 : SSID, APK, ACL, DNS, CA | 5 : same pattern as family access | 175+150+60+75 | **460** |
| guest-only. Guest SSID, no tailnet | 2 : no PopCornTV | 5 : isolated | 3 : SSID + firewall on the box | 4 : isolation, no tailnet pattern | 70+150+60+60 | **340** |
| main-ts. Main Wi-Fi + Tailscale | 5 : all reachable | 1 : LAN access to SSH 22, 1883, box UI, every device; adbd 5555 open to the LAN | 4 : APK only | 4 : Tailscale pattern, no isolation | 175+30+80+60 | **345** |

Sensitivity (others renormalised proportionally): main-ts passes guest-ts only below security ≈ 2 (at 0: main-ts 450 vs guest-ts 443). The framing session showed guest-ts 470 / main-ts 425 / main-wifi-caddy 240 with a "nomadic" criterion and main-ts's security at 3; nomadic is already covered by D2 and that security cell was wrong.

Box under C, same criteria: guest SSID + tailnet = 5 (all reachable) / 4 (vendor firmware isolated from the LAN, ACL on the tailnet) / 3 (SSID, Tailscale, ACL, DNS, CA) / 5 (same pattern) → 175+120+60+75 = **430**; main Wi-Fi + tailnet = 5 / 3 (box sees SSH 22, 1883, every device) / 4 / 4 → 175+90+80+60 = **405**. Reasons for main Wi-Fi: need 5 (all reachable), simplicity 4 (no second SSID), coherence 4 (Tailscale pattern, no isolation). Sensitivity (others renormalised proportionally): main Wi-Fi passes guest only below security ≈ 7 (at 5: 443 vs 441). Guest SSID; the throughput gate of D3 applies to the box.

### D4 matrix under C (works anywhere 30, family UX 25, hardware risk 20, cost 15, depends on an unverified fact 10)

Scored with the IR blaster as a given (needed for power-off and HDMI selection under C) and the box carried with the projector when nomadic.

| Option | Anywhere | Family UX | HW risk | Cost | Unverified | Points | Total |
|---|---|---|---|---|---|---|---|
| plug. Smart plug + auto-boot, IR off | 3 : same blaster plus a plug and Wi-Fi for it | 4 : one scene | 2 : hard cut at every off | 4 : plug owned, blaster anyway | 1 : auto-boot untested | 90+100+40+60+10 | **300** |
| manual. Manual on, IR off, HA follows | 3 : the blaster travels with the box anyway | 2 : power on first, then select the input | 5 : nothing forced | 3 : blaster anyway, nothing extra | 2 : "HA follows" needs the box to report state after a cold boot | 90+50+100+45+20 | **305** |
| ir. IR on and off | 3 : same kit as plug | 4 : one scene, input selected by IR | 5 : the remote's own commands | 3 : blaster anyway, nothing extra | 3 : IR power-on from full off untested, codes to capture | 90+100+100+45+30 | **365** |

Sensitivity (others renormalised proportionally): ir stays on top at every hardware-risk weight; plug leads only on cost and passes ir only if cost weighs ≈ 49 or more (at 50: plug 341 vs ir 338). plug becomes a fallback only if IR power-on fails and auto-boot is confirmed. The session matrix (scored for an Android projector under strategy A: 330 / 395 / 290, manual on top) is superseded; its plug option scored "no fixed hardware" and "nothing to buy", both false once the blaster is required.

## 5. Milestones and bricks

Steps marked **[A]** apply only if strategy A is reinstated; everything else is strategy C or common.

### v0.1 — Reconnaissance (hardware required; common to A and C)

Order is mandatory: the firmware joins no network before the dump and the egress capture. Under C the dump (step 3) is kept as the backup that v3 (Linux study) requires and for its learning value; A can pass C only if step 5 lowers C (§3); then steps 4 and 6 become mandatory before ADR 0003 is confirmed. Under C, run step 5 (its network-free items) before step 4 (unless the OTG branch of step 2 has already run step 4); if step 5 lowers C, run step 4 and then step 6.

1. **Safety** (runbook 00): mains unplugged, primary bulk capacitor of the off-line supply (~325 V DC) measured and discharged, then the ~27 V LED rail, ESD; bench 5 V on `5V_IN` preferred for powered steps; never power the board with the heatsink lifted or the fan unplugged; eye protection with optics open; USB A-to-A cable with VBUS cut; re-pad the heatsink if lifted.
2. **#2 Identification, no network**: About screen (Android version, build fingerprint, SoC), `getprop ro.build.type`, `ro.adb.secure`, `ro.product.board`; Developer options present; which USB port is OTG (one USB-A on the board). Photos of the About screen are redacted (serial, MAC). **Branch: if the USB-A port is not OTG**, every USB dump path of step 3 is impossible; network ADB then needs a firmware "network debugging" toggle or Android 11+ wireless debugging with a pairing code (`adb pair`); if available, the first network join happens on the PC hotspot of step 4 with Wireshark already running and the dump goes over network ADB; if neither is available, the dump is a recorded no-go.
3. **Dump feasibility and dump (#3)**, in order of preference, each with pass/fail: Loader mode (`adb reboot loader` or RECOVERY button + USB) + `rkdeveloptool rl` if Rockchip (OS not running: whole-image hashes comparable); `adb root` + `dd` with stderr discarded (live read: hashes compared per read-only partition); Maskrom + matching `rkbin` loader; else partial backup (APKs via `pm path` + `adb pull`, `/sdcard`, readable parts of `/system` and `/vendor`). Verification: two independent reads with identical SHA-256 (whole image, or per read-only partition on a live read), GPT parsed and matching, `super` unpacked with `lpunpack` and mounted read-only. **No write to the eMMC in v0.1**; a write-back restore needs its own ADR and a proven loader. Hashes in `dumps/README.md`; the dump is stored encrypted with an off-laptop copy; only hashes are ever shared (it holds per-unit keys). Go/no-go: without a verified full dump, no modification beyond `pm uninstall --user 0` for the whole project.
4. **[A, or C if step 5 fails] Egress capture (#5)**: first network contact on a PC hotspot that is really isolated (separate WAN, `tailscale down`, drop rules to private ranges; runbook 03) with Wireshark, 30 min idle + 30 min use; list every destination. APK inventory with hashes and VirusTotal lookups; decision per vendor package. Hash-only uploads. Under C it feeds the research record, the §3 re-score if step 5 lowered C (and the OTG branch of step 2 if the dump needs network ADB); under C the projector's only network contact is this isolated hotspot session (runbooks 03 and 05), forgotten afterwards.
5. **Power states**: what "off" is (draw in W; standby with Wi-Fi alive? [A, or after step 4]), `KEYCODE_WAKEUP` from standby [A], auto-boot on mains return, IR codes of the remote captured (power, volume, input; remote confirmed IR, not Bluetooth-only), IR power-on from full off, IR input selection from the vendor launcher, whether the firmware boots to or remembers the HDMI input.
6. **[A, or C if step 5 fails] Cold-boot checks (runbook 05, same hotspot as step 4)**: network ADB on 5555 alive after a cold boot without USB; if not, the schaumann-byte helper app or an equivalent keep-alive.
7. **UART** (optional): level measured with a meter before wiring, baud probably 1 500 000 if Rockchip (RK33xx/RK35xx), 115 200 on other vendors and older parts; boot log captured through the redaction script.
8. **[A] Network check (issue to file; cross-repo dependencies below)**: guest SSID on the operator router; Tailscale APK pinned (source, version, SHA-256); minimum Android version; VPN consent dialog present; whether a user CA forces a screen lock; tagged pre-auth key; tailnet ACL state confirmed in the admin console; ACL applied before the projector joins; name resolution and CA trust; Tailscale up after a cold boot; throughput gate of D3 measured from the projector. **Under C the same checklist runs on the box in v0.2**, except the APK/firmware items.

**Cross-repo dependencies (bb-homelab)**, blocking step 8 under A and v0.2 under C: (a) tailnet ACL policy, default-deny with `tag:projector` or `tag:media-box`; (b) Caddy `remote_ip` matcher returning 403 to that client on every host but PopCornTV; (c) name resolution: a resolver on the Pi for Tailscale split DNS (none under `services/` today, to confirm) or the public domain of bb-homelab #28 with ACME.

Acceptance v0.1: SoC and build identified; dump verified or the no-go recorded; egress list reviewed (if captured); pass/fail recorded for: OTG port, each dump path, standby state, auto-boot on mains return, IR codes captured, IR power-on from full off, IR input selection, HDMI input remembered; under A also `KEYCODE_WAKEUP`, ADB 5555 after cold boot, Tailscale after cold boot, CA install without screen lock, throughput gate. ADR 0003 (C) confirmed, or, if step 5 lowered C, steps 4 and 6 run and §3 re-scored before it is confirmed; ADR 0005 frozen.

### v0.2 — Home Assistant (strategy C)
- **Box choice by MCDA** (issue; the ACL of dependency (a) lists the ports HA needs per candidate, e.g. 6466/6467 Android TV Remote, 5555 `androidtv`, 8009 cast, and the host is set by hand since mDNS discovery does not cross the tailnet): candidates a Raspberry Pi with a browser kiosk, an Android TV / Google TV stick (HA core `androidtv` and Android TV Remote integrations), a Chromecast-class device. **Pass/fail gates before the matrix**: (1) runs Tailscale (rules out cast-only devices); (2) trusts the Caddy root in the player actually used (Android TV / Google TV ship WebView-based browsers that ignore a user CA by default, and may have no certificate-install screen — to verify), **or** bb-homelab #28 (public domain + ACME) is delivered first, which makes #28 a likely dependency under C. Criteria: HA integration maturity, codecs, remote-control UX, cost, power-on by HA.
- Box on the guest SSID + tailnet with the D3 rules (tag, ACL, Caddy 403, DNS, CA, throughput gate); the three cross-repo bricks delivered first.
- Entities: the box's `media_player` (state, power, volume, app launch, remote keys; volume lives here since audio travels over HDMI); the projector's power and HDMI input by IR (ESP-based or Broadlink blaster, codes from v0.1 step 5), or a smart plug + IR off if IR power-on failed and auto-boot was confirmed (D4).
- "Movie night" scene started by a **user tap**: lights, box on, PopCornTV launched, notification.
- Latency NFR: ≤ 60 s from power-on to HA `on` for the box, measured.
- **[A]** instead: `androidtv` / schaumann-byte / own MQTT bridge by MCDA; ADB keep-alive conditional on the Android version.

Acceptance v0.2: every entity above works from a cold boot three times in a row; latency NFR measured; projector power and HDMI input driven at least once by HA.

### v0.3 — PopCornTV on screen (strategy C)
- Player measurement (D6) on the box: dropped-frame counts, Pi CPU load (the Pi 5 has no hardware encoder: HLS transcode is a Pi risk), box thermal state over 2 h; MCDA; implementation.
- Dedicated non-owner PopCornTV account for the box (ADR-0010 allows it); persistent session. If PopCornTV lacks a needed feature, an issue is filed as `benoit-bremaud/popcorntv#<n>` at v0.3 start.
- **[A]** instead: measurement on the projector, debloat + launcher runbook (D5) with the undo command per package.

Acceptance v0.3: 1080p direct play for 2 h with < 0.5 % dropped frames and no thermal throttling; login survives a reboot; family can start a film with the remote only.

### v1.0
- Signed tag; "reproduce at home" guide; dry run of every runbook by someone other than the author (or, failing that, the owner on a fresh machine and account) following only the repository: zero blocking deviations. The third-party claim of section 1 covers exactly what this dry run covers.
- QA pass limited to what applies (HA dashboard via `qa-browser-check`; projector and box flows by the runbook checklist).

Acceptance v1.0: from a cold projector, movie night in ≤ 3 remote presses + 1 HA tap (the tap powers the projector by IR, selects HDMI and starts the box; the presses pick the film); all of v0.1–v0.3 acceptance green; traceability matrix complete.

### Later
- **v2** playback driven from the phone (PopCornTV "playback target").
- **v3** Linux on the board: study only, opened once the SoC and a loader are known.
- HA extras: Bluetooth audio output, sleep timer, now-playing from the PopCornTV API, energy via smart plug, on-screen toast, ECO/brightness.

## 6. Reproducibility and publication

- The repository is public. **NFR-redaction**: serial number, MACs, SSID/PSK, Tailscale keys and IPs are removed from every log, screenshot and `getprop` output before staging; captures go through `tools/redact.sh` (covered by a test); a pre-commit check greps MAC and serial patterns; `uart-logs/` is only ever written through the script.
- What a third party brings: a unit that meets the identification criteria of runbook 01 (other boards are unsupported, the H713 material is background only), a PC with `adb`, a 3.3 V USB-UART adapter and a VBUS-cut USB A-to-A cable (recon only: the v1.0 end state under C needs neither), Home Assistant, Tailscale, an HDMI box that passes the v0.2 gates, an IR blaster, and a PopCornTV (or any HTTPS web player) instance. Everything else is in the runbooks.

## 7. Conception dossier skeleton (created on validation)

```
docs/architecture/
  README.md, glossary.md, traceability-matrix.md
  decisions/0001-record-architecture-decisions.md
            0002-adopt-hybrid-up-iconix-method.md
            0003-drive-the-projector-through-a-trusted-hdmi-box.md   (C; A and B rejected; review clause)
            0004-put-the-media-box-on-the-guest-ssid-and-the-tailnet.md
            0005-control-the-projector-power-by-ir.md                (frozen after runbook 04)
  specs/projector-integration-requirements.md, projector-integration-risks.md, security-redaction.md
  diagrams/projector-integration/01-use-case.md, 02-sequence-movie-night.md, 03-component.md,
                                 05-state-projector.md, 08-deployment-home.md
  runbooks/00-safety.md, 01-identification.md, 02-dump.md, 03-egress-capture.md, 04-ir-capture.md
```

Created on 2026-11-06 after the owner's go. Names follow the `conception-naming` convention (fixed diagram slots, ADR titles as verb phrases), which supersedes the draft names of revision 3.

## 8. Changelog of the reviews

- Self-review (10 points) and adversarial round 1 (8 Must, 14 Should, 5 Nice) applied in revision 2: HDMI fact corrected and strategy re-run; body and notes merged; no eMMC write in v0.1 with a defined read-back; recon order "identify → dump → egress → ACL → tailnet"; fallback D rescored and gated by derogation; DNS and CA trust as v0.1 items; redaction NFR; thresholds and cold-boot checks; vendor marked unknown; dump paths ordered; latency NFR end to end; firmware-level malware model; Tailscale operations; Caddy 403 and dedicated account; Kodi dropped unless scoped; acceptance per milestone; matrices recorded; issues to file; safety; revocation conditional; scene on user tap; wake tested; state and sequence diagrams; dump custody; undo command.
- Adversarial round 2 (verify 6113759) applied in commit 7114cdc: research docs corrected by dated notes; OTG branch; Tailscale cold-boot check after the join; §3 arithmetic corrected (345/285/380); D1 follows the top score; D3 and D4 matrices with points; cross-repo dependencies blocking; ACL state marked inferred; third-party scope narrowed; dry run by a non-author; acceptance list named; CA/screen-lock check.
- Adversarial round 3 (verify 7114cdc) applied in revision 3: decision rows aligned with the matrices (D3 245/460/340/345, D4 330/395/290); **owner validated the weights → C is the strategy**; plan restructured with a C track (box choice, IR, box on the tailnet) and [A] markers; ADR 0003 renamed neutral; OTG branch names the enabling method and the no-go; v0.1 facts that could move a §3 cell listed; D4 "unverified" cell for B corrected to 2 with a reason; reasons and a sensitivity line on D3 and D4; decisions table no longer broken by the matrices; §2 ACL fact marked inferred; revision and section numbers fixed.
- Adversarial round 4 (verify 724b6d4) applied in revision 4: D4 re-scored under C with the IR blaster as a given (IR on/off 365, smart plug 300 (330 before round 5), manual 305; validated by the owner on 2026-11-06); D1 and D5 wording corrected (network contact in v0.1 step 4, vendor UI for HDMI selection); v0.1 facts that can lower C listed and tested in step 5; egress capture marked [A, optional under C], dump justified as the v3 backup; box pass/fail gates (Tailscale, CA trust or #28) before the box MCDA; box network scored (guest SSID 430 vs main Wi-Fi 405); D3 and D4 sensitivity numbers recomputed; volume on the box; recon-only tools marked in §6; ADR 0005 renamed.
- Adversarial round 5 (verify c168310) applied in revision 5: §5 preamble aligned with §3 (steps 4 and 6 mandatory under C if step 5 lowers C); D4 A "anywhere" 3 (300) and sensitivity recomputed; box matrix reasons and sensitivity; HDMI-memory condition tied to IR input selection; ACL ports per box candidate.
- Adversarial round 6 (verify 2b57e65) applied in revision 6: under C step 5's network-free items run before step 4, the Wi-Fi standby item moved after step 4; box and D4 sensitivity lines recomputed; header lists D4 as validated.
- Commit dcbb9c8: OTG-branch exception added to the v0.1 order (round 7 Nice).
- Pre-push review round 1 (adversarial + doc audit on 43949d8) applied in revision 7: sub-decision options renamed with words (D3, D4) to stop colliding with strategies A/B/C; safety step names the primary capacitor; dump verification per path; hotspot isolation; runbook 05; derogation ADR unnumbered; #3/#5 cited.
- Pre-push review round 2 (verify 916df42) applied: option letters removed from §4 (D3, D4 rows and sensitivity lines), step 4 wording under C, IR input selection in step 5 and the acceptance list, operator router naming, D3 box variant `guest-ts` validated by the owner (ADR-0004 accepted).
- Out of scope, pre-existing in bb-homelab: SSH `ALLOW Anywhere` on the LAN (AUDIT R13).
