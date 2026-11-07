# Board identification from the 2026-11-05 teardown photos

What the 119 teardown photos (taken 2026-11-05, read on 2026-11-06) establish about our own unit. Readable markings only; the main SoC is still unidentified.

## Verdict

**This is not an Allwinner H713 board; the SoC vendor is unknown.** The H713 has HDMI input built in and would not need the Rockchip RK621 bridge found here. The bridge, the `RST/RECOVERY` button and the 24 MHz crystal make Rockchip the likely vendor, but a standalone RK621 can sit next to any TV-box SoC that only has HDMI output (Allwinner H313/H618, Amlogic, Rockchip); the About screen or the USB vendor id (`2207` Rockchip, `1f3a` Allwinner, `1b8e` Amlogic) settles it. *(Verdict softened 2026-11-06 after review; the first wording claimed Rockchip.)* Everything in the desk research that is specific to the H713 (YuujiLab dump method, FEL, the well0nez mainline port, anti-rollback bypass) does **not** apply to this unit. The Home Assistant side (wireless ADB) is unaffected.

## Readable components

| Item | Marking | Meaning |
|---|---|---|
| HDMI bridge | `Rockchip RK621 HAAUUR650 2523 29V0105 000` (QFN56) | HDMI 1.4/2.0 in → MIPI DSI / LVDS out with scaler; drives the LCD from the HDMI input. Date code 2025 week 23. |
| Memory | `SAMSUNG KMQ7X000SA-B315 708` | eMCP: 2 GB LPDDR3 + 16 GB eMMC in one package; matches the label "2+16". |
| Main SoC | not visible | Under the finned aluminium heatsink on the top side; no photo shows it. |
| Audio amplifier | `NS4152S 2509Y1` (SOIC-8) | Class-D speaker amplifier. |
| Audio codec (probable) | `ES7148 Z8WM04` | Everest Semi DAC, next to the recovery button. |
| Crystal | `JWT 24.000C12` | 24 MHz, the usual Rockchip reference clock. |
| Wi-Fi/BT | unmarked metal shield, antenna pad beside it | Module unknown. |
| Regulators | SOT `AHAAd`, `A2SHB`, `LPS 1FFb4`; inductors `1R0`, `2R2` | Buck converters. |
| PCB | silkscreen `20250318`; `94V0 MJS-M E525332`; ink `XT250723` | PCB design date 2025-03-18, UL file E525332, batch 2025-07-23. |
| Power board | `AC100-240V DC24-29V-27V 1.1A / 5V 2A, 2025.07.03-DL` | Separate PSU board: ~27 V for the LED, 5 V for the mainboard. |
| LED module | `YUN 2226-4 9C2B` | White LED on heatsink, +/- pads. |

## Connectors and labels (silkscreen)

`POWER` (button), `+SPK-`, `HDMI` (input), `EAR` (3.5 mm), `USB` (USB-A), `+FAN-`, `5V_IN` (`5V 5V GND GND EN`), `MIPI/LVDS LCD` (FFC), `IR` (`IR GND VCC`, 3-pin header), `TF` (microSD), `RST/RECOVERY` (button), `TX RX GND` (three round pads next to the IR receiver, bottom side).

## Sticker (bottom side, translated)

`客户订单: YDF10020250717-02` — customer order 2025-07-17-02
`物料编码: DZ-00040` — material code
`软件: 定制法语` — software: custom French build
`XTSM 2+16 07.25` — 2 GB + 16 GB, 2025-07

## Access paths implied

- **UART**: `TX RX GND` pads, bottom side. If the SoC is Rockchip, the console probably runs at 1 500 000 baud (RK33xx/RK35xx); 115 200 on other vendors and older parts. 3.3 V TTL expected; verify the level with a meter before wiring.
- **Recovery / Loader / Maskrom** (if Rockchip): the `RST/RECOVERY` button held at power-on with a USB link normally gives **Loader** (Rockusb) mode, which runs the board's own loader and already allows `rkdeveloptool rl` reads without any `rkbin` blob; `adb reboot loader` reaches it too. **Maskrom** (BootROM, software-unbrickable) usually needs an eMMC CLK/D0 short and a matching loader. *(Corrected 2026-11-06 after review: the first wording called the button the Maskrom entry.)*
- **USB-A port**: on Rockchip boards the OTG port is usually the USB-A next to the SoC; which port enumerates in Maskrom is to be tested.
- **ADB**: identical to the H713 plan; wireless ADB is what Home Assistant uses.

## Still unknown (needs the hardware)

1. Main SoC part number: *Settings → About* (build fingerprint contains the chip, e.g. `rk3528`) or lift the heatsink.
2. Android version and build date; whether Developer options exist.
3. Wi-Fi/BT module under the shield.
4. Which USB port is OTG and whether Maskrom enumerates (`lsusb` shows `2207:xxxx`).
5. UART logic level and baud.

## Photos kept in `photos/`

`2026-11-05-mainboard-top.jpg`, `-mainboard-bottom.jpg`, `-rk621-hdmi-bridge-and-recovery-button.jpg`, `-samsung-emcp-kmq7x000sa.jpg`, `-uart-pads-tx-rx-gnd.jpg`, `-order-label-2plus16.jpg`, `-wifi-shield-crystal-fan.jpg`, `-power-board-and-lcd.jpg`, `-optics-open.jpg`. The full 119-photo set stays outside the repository.

## Sources

- <https://www.rock-chips.com/a/en/products/RK6_Series/2025/1208/2128.html> — RK621 product page
- <https://rockchip.fr/RK621%20datasheet%20V1.0.pdf> — RK621 datasheet
- <https://github.com/rockchip-linux/rkdeveloptool> — Maskrom / loader tool
