# Runbook 01 — identify the board without network (UC4, REQ-5)

Precondition: runbook 00 ticked. The projector has never been connected to any network.

## Steps

1. Power on. First action: Wi-Fi **and Bluetooth off** in Settings (or airplane mode if offered), skip the Wi-Fi setup, check that no saved network exists; the firmware or an open-network auto-join could otherwise connect on its own.
2. *Settings → About*: photograph Android version, build number / fingerprint, model. **Photos**: crop or blur the serial and MAC by hand, strip EXIF (`exiftool -all= <file>`), keep the original in `photos/raw/`. **Text**: pipe through `tools/redact.sh` (brick #7; until it exists, redact by hand against `specs/security-redaction.md`).
3. Enable Developer options (tap the build number) and USB debugging.
4. Connect the VBUS-cut A-to-A cable to the USB-A port; on the PC: `adb devices` and `lsusb`.
   - Vendor id `2207` → Rockchip; `1f3a` → Allwinner; `1b8e` → Amlogic. Record it.
5. Properties (toybox `getprop` takes one name at a time):

   ```bash
   for p in ro.build.type ro.adb.secure ro.product.board ro.hardware ro.build.fingerprint; do printf '%s=' "$p"; adb shell getprop "$p"; done | tools/redact.sh
   ```

   Until `tools/redact.sh` exists (#7), drop the pipe and redact the output by hand before pasting it anywhere (`specs/security-redaction.md`). `ro.serialno` is not read: REQ-5 does not need it.

6. Record whether a firmware "network debugging" toggle or Android 11+ wireless debugging (`adb pair`) exists.

## Pass/fail to record

| Check | Result |
|---|---|
| SoC vendor from USB id or fingerprint | |
| Android version, build type (`user` / `userdebug`) | |
| USB-A port is OTG (device enumerates) | |
| Developer options present | |
| Network ADB method available (toggle / `adb pair` / none) | |
| Wi-Fi and Bluetooth off, no saved network | |

## Branch

USB-A not OTG → runbook 02's USB paths are impossible; if a network ADB method exists, the dump goes over network ADB on the hotspot of runbook 03 with the capture already running; otherwise record the dump no-go in `dumps/README.md`.
