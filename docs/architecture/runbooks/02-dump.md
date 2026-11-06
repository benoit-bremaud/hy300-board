# Runbook 02 — read-only firmware backup (UC5, REQ-6, NFR-6)

Precondition: runbook 01 done, OTG port confirmed (or the network-ADB branch). **Nothing in this runbook writes to the eMMC.** Preferred path: a read while the OS is **not** running (Loader mode), because a live read changes between two passes.

## Paths, in order of preference (stop at the first that passes)

1. Rockchip only, OS not running: `adb reboot loader` (or hold `RST/RECOVERY` at power-on with USB connected) → `rkdeveloptool ld` lists the device → `rkdeveloptool rl 0 <sectors> dump-1.img` (sector count from `rkdeveloptool rfi` / the GPT).
2. Live read: `adb root` then `adb exec-out "dd if=/dev/block/mmcblk0 bs=4M 2>/dev/null" > dump-1.img` (userdebug builds, or permissive SELinux with a readable block device). `2>/dev/null` is required: `exec-out` merges `dd`'s stderr into the stream and would corrupt the image.
3. Rockchip only: Maskrom (eMMC CLK/D0 short at power-on, runbook 00) + the matching `rkbin` loader → `rkdeveloptool db <loader>` (loads the loader into RAM only, no eMMC write) then `rl`. Last resort.
4. Partial backup: `pm list packages -f` → `adb pull` of every APK path; `/sdcard`; readable parts of `/system` and `/vendor`.

## Verification

- Second independent read `dump-2.img`.
- Path 1 or 3 (OS not running): `sha256sum dump-1.img dump-2.img` identical.
- Path 2 (live): whole-image hashes **will differ** (`userdata`, `metadata`, `misc`, logs change). Compare per partition: `sgdisk -p dump-1.img` for the table, then `dd`-extract and hash the read-only partitions (`boot*`, `vbmeta*`, `super`, `uboot`/`trust`, key/`private` partitions); identical hashes there pass; record the expected difference on the writable partitions.
- GPT parsed on both reads with the same table; partition names recorded.
- `lpunpack` on the `super` image; `mount -o ro,loop` of `system`.
- `dumps/README.md`: file, date, method, size, SHA-256 (per partition for a live read). Hashes only; never the image.

## Custody

Image encrypted (`age` or `gpg`) with an off-laptop copy; it holds per-unit keys (HDCP, MAC, serial). Only hashes are ever shared.

## Go/no-go

Without a verified dump (whole image, or every read-only partition for a live read), no modification beyond `pm uninstall -k --user 0` is allowed for the whole project (and under ADR-0003 none is planned).
