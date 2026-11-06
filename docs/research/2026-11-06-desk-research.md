# Desk research — what the community knows about the HY300 (2026-11-06)

What is already known about the HY300 projector family before touching our own unit: hardware, firmware, access paths, existing integrations and the mainline Linux port. Every claim comes from the sources listed at the end; nothing here is verified on our board yet.

## 1. Hardware variants (the first thing to identify)

"HY300" is a white-label name covering several boards. The variant decides every later step.

| Sold as | SoC | DRAM | Android | Notes |
|---|---|---|---|---|
| HY300 (2023) | Allwinner H713 (sun50iw12p1) | 1 GB DDR3 | 11 | 8 GB eMMC, 720p panel |
| HY300 Pro (2024) | H713 | 1 GB DDR3 636 MHz | 11 | Linux port: in test |
| HY300 Pro+ (2025) | H713 | 1 GB LPDDR3 | 11 | Linux port: profile only |
| HY300PRO-G-EU (2025-05) | H713 | DDR3 792 MHz | 11 | Linux port: verified |
| HY300 Pro / Ultra (2025+) | Allwinner H726 | 2 GB | 14 | Different SoC, different firmware, no Linux port |

Common H713 facts: quad Cortex-A53, Mali-G31 MP2, Toshiba/Kioxia 8 GB eMMC 5.1 (`/dev/block/mmcblk0`), kernel 4.9.170, 32-bit userspace (`armeabi-v7a`), Wi-Fi/BT AIC8800. **HDMI is an input** on every variant: an external box can feed the panel; the projector cannot output to another screen. *(Corrected 2026-11-06 after review; an earlier wording said the opposite. See `2026-11-06-framing-and-plan.md` §2.)*

How to tell: *Settings → About* (Android version, build fingerprint such as `Allwinner/h713_tuna_p3/...`), or the SoC marking on a board photo.

## 2. Access paths on stock firmware

- **ADB** works unrooted. SELinux is permissive and shell uid 2000 can read block devices, so a full eMMC dump needs no root:
  `adb exec-out "dd if=/dev/block/mmcblk0 bs=1M" > emmc_full.img` (expected 7 818 182 656 bytes on the 8 GB eMMC).
- **Wireless ADB** (port 5555) is what every Home Assistant integration relies on. Android revokes wireless-debugging keys unused for 7 days; the fix is setting the ADB authorisation timeout to 0.
- **UART**: pads next to the SoC, 3.3 V TTL, 115200 8N1, cross TX/RX, GND only. Stock U-Boot 2018.05 with `bootdelay=0` (no interrupt window unless the env is patched).
- **FEL** (Allwinner BootROM recovery, USB id `1f3a:efe8`): hold Volume+ (or the reset/FEL pad) at power-on. FEL lives in ROM and cannot be bricked from software; it is the way back from any flash.
- **Root**: Magisk, by pulling `boot_a` over ADB, patching it, flashing with fastboot. Stable on Android 11.

Partition layout: 26-partition GPT, virtual A/B where only `_a` slots are real (`_b` are zero-filled; booting `_b` hard-bricks). `super` holds `system`, `vendor`, `product`. `private` and `Reserve0` hold per-unit keys (HDCP, MAC, serial) and must be backed up.

Flashing pitfalls: anti-rollback via secure storage (`pst write firmware_version ...` from U-Boot to bypass); USB burn-in only from a cold power cycle; batches change layouts without notice.

## 3. Security note: malware in older H713 firmware

Several reports describe pre-installed malicious APKs in older H713 stock firmware. Magcubic's official statement says units on the H726 chip do not contain it. Consequence for us: the stock firmware is not trusted; the projector stays on an isolated network segment (bb-homelab per-backend network pattern) whatever the chosen strategy, and the dump must be scanned before anything is reused.

## 4. Home Assistant: what already exists

- **`androidtv` core integration** (ADB-based): power, volume, key presses, app launch; one ADB client at a time; built-in Python ADB preferred over an ADB server.
- **schaumann-byte/hy300-projector-homeassistant**: a dedicated custom integration for HY300 Pro+ (tested on the H726 / Android 14 variant, HA ≥ 2025.3). Combines the vendor's local "Lumina Go" API, IR for power-on (the device is off, ADB cannot wake it) and ADB in a side container. Ships a tiny app that keeps wireless ADB alive and fixes the boot launcher. Documents the 7-day key revocation issue.
- Power-on is the hard part everywhere: a powered-off projector answers nothing on the network, so power-on needs IR (Broadlink or ESP-based blaster) or a smart plug plus auto-boot.

## 5. Software on the Android side

- Stock launcher "LuminaOS" with hard-coded streaming buttons; replaceable by Molla or Projectivy (the usual Android TV launchers), bloat removable over ADB.
- Kodi and VLC run; the CPU/RAM (1 GB) limit heavy apps. Autostart of a chosen app at boot is unreliable with generic tools (launch-on-boot fails; the schaumann-byte helper app handles it).
- Stock browser quality is unknown; it decides whether PopCornTV's web player can run directly. To measure on the device.

## 6. Mainline Linux port (well0nez/allwinner-h713-linux)

Far more advanced than a research project: Linux 6.18, open boot chain (U-Boot SPL, TF-A, U-Boot), no vendor blobs in the repo (the installer extracts them from the user's own dump), FEL-based installer with `identify | dump | install | restore | restore-stock`.

Verified on HY310, HY300PRO-G-EU and L018; HY300 Pro in test; HY300 Pro+ profile only. Working: HDMI input 1080p60, HDCP with the unit's own keys, audio, IR and Bluetooth remote, keystone on the GPU, autofocus, Wi-Fi/BT, **Kodi, labwc desktop and Firefox with hardware video decode** (H.264, HEVC, VP8, VP9), a settings web panel on port 8080, SSH. Known gaps: no real standby (2.9 W off state), lamp flicker on some units, older HY300 (Pro) IR not working.

Why it matters for us: a Linux projector is a first-class homelab citizen (SSH, systemd, MQTT client, Firefox kiosk for PopCornTV, Kodi) instead of an untrusted Android box reached over ADB. The entry ticket is a full dump (also the only way back) and the right board profile.

## 7. Consequences for the strategy decision

The 2026-11-06 MCDA scored "Linux mainline" at 245/500 on the assumption of a research-stage port with uncertain video. The research shows a port with hardware video decode, Kodi/Firefox and an unbrickable FEL recovery. That assumption was wrong, but the point became moot the same day: the teardown photos rule out the H713 and make Rockchip the likely but unproven vendor *(wording corrected 2026-11-06 after review, see the plan §2)* (see `2026-11-06-board-identification.md`). Sections 1, 2 and 6 above are kept as background on the H713 family only; sections 3, 4 and 5 (malware, Home Assistant, Android software) still apply. The matrix was re-run the same day with a security criterion and the corrected HDMI fact: see `2026-11-06-framing-and-plan.md` §3 (strategy C, trusted HDMI box, 380/500).

## Sources

- https://github.com/YuujiLab/HY300-H713-Research — dump, partition layout, UART/FEL, root, anti-rollback
- https://github.com/well0nez/allwinner-h713-linux — mainline port, board status table, installer, UART doc
- https://github.com/shift/sun50iw12p1-research and https://github.com/srgneisner/hy300-linux-porting — earlier research-stage ports
- https://github.com/schaumann-byte/hy300-projector-homeassistant — dedicated HA integration (Lumina Go + IR + ADB)
- https://www.home-assistant.io/integrations/androidtv — core ADB integration
- https://gist.github.com/probonopd/3ad6b7777caea1503f00d5fe7710ad06 — launcher replacement, debloat, autostart notes
- https://xdaforums.com/t/beta-android-tv-and-root-tutorial-for-allwinner-h713-based-projectors.4744020/ — Android TV + root tutorial (beta)
- https://www.magcubic.com/es/pages/official-statement-regarding-certain-apk-files-on-magcubic-hy300-series-products — vendor statement on malicious APKs
- https://androidpctv.com/firmware-magcubic-hy300-allwinner-h713/ and https://androidpctv.com/firmware-magcubic-hy300-pro-ultra-h726/ — H713 vs H726 firmware
