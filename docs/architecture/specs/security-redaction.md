# Security — redaction of captures (NFR-2)

What must never reach the public repository **or a public issue**, where it comes from, and how it is removed.

## Sensitive fields

| Field | Appears in | Pattern |
|---|---|---|
| Serial number | About screen, `getprop ro.serialno`, UART boot log, dump metadata | `ro.serialno=…`, `[A-Z0-9]{10,}` next to "serial" |
| MAC addresses (Wi-Fi, BT) | `getprop` (`ro.boot.wifimac`, colon, dash or separator-less forms), UART log, Wireshark export, HA device page | `([0-9A-Fa-f]{2}[:-]?){5}[0-9A-Fa-f]{2}` in a MAC context |
| IPv6 addresses | Tailscale (`fd7a:115c:a1e0::/48`), EUI-64 link-local (embeds the MAC) | `fd7a:115c:a1e0:[0-9a-f:]+`, `fe80:[0-9a-f:]+` |
| SSID / PSK | `wpa_supplicant.conf` in a dump, screenshots | by name |
| Tailscale pre-auth key, node key | ACL work, box setup logs | `tskey-`, `nodekey:` |
| IP addresses (LAN, tailnet) | logs, Caddy config, HA | `\b\d{1,3}(\.\d{1,3}){3}\b` **in an IP context only** (after `inet`, `src`, `dst`, `addr`, `ip`): version strings such as a firmware `9.0.0.221` must survive |
| EXIF (device model, timestamps) | every photo and screenshot | `exiftool -all=`; the nine committed photos were checked: no GPS |
| Per-unit keys (HDCP, vendor) | dump partitions | never extracted; the dump itself is never committed |

## Data flow

Capture (device, PC) → `tools/redact.sh` (sed rules per pattern above, placeholders `<SERIAL>`, `<MAC>`, `<IP6>`, `<IP>`) → paste into an issue, or `uart-logs/` / `docs/` → pre-commit grep (MAC and serial patterns) → commit. Issue pastes get no pre-commit check, so the runbooks pipe every output through the script explicitly. Raw captures stay in `photos/raw/`, `dumps/`, `*.pcap*` or outside the repository; `.gitignore` covers them.

## Verification

- `tools/redact.sh` (brick #7) has a test with one fixture per pattern, including the version-string counter-example.
- The dry run of REQ-8 includes `gitleaks detect` on the full history and an EXIF check on committed images.
