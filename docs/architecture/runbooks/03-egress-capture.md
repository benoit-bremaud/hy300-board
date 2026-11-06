# Runbook 03 — egress capture on an isolated hotspot (NFR-1, R7)

When: strategy A, the OTG branch of runbook 01, or ADR-0003's review clause (runbook 04 lowered C). Under strategy C otherwise optional. This is the projector's **only network contact under C**; if the review clause fires, the cold-boot checks of runbook 05 reuse the same hotspot before it is forgotten.

## Setup — the hotspot must really be isolated

A PC hotspot that shares the PC's own connection NATs the projector onto the home LAN (and the tailnet when `tailscale0` is up). So:

1. Uplink through a **separate WAN** (phone tethering on the PC), with the PC's home Wi-Fi/Ethernet disconnected and `tailscale down`.
2. Hotspot interface with drop rules toward private ranges (adapt the interface name):

   ```bash
   sudo nft add table inet hy300-capture
   sudo nft add chain inet hy300-capture forward '{ type filter hook forward priority 0; policy accept; }'
   for net in 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 100.64.0.0/10; do sudo nft add rule inet hy300-capture forward iifname "<hotspot-if>" ip daddr $net drop; done
   sudo nft add rule inet hy300-capture forward iifname "<hotspot-if>" ip6 daddr fd7a:115c:a1e0::/48 drop
   ```

   A dedicated table leaves any existing firewall (`inet filter` on Debian) untouched. Cleanup after the capture: `sudo nft delete table inet hy300-capture`.

3. Pass/fail before the projector joins: from a phone on the hotspot, the Pi, the operator router's admin page and any tailnet address do **not** answer.
4. Hotspot name and key are throwaway and never committed.
5. `tshark -i <hotspot-if> -w egress.pcapng` (or Wireshark) running **before** the projector joins. The capture file stays outside the repository (`.gitignore` covers `*.pcap*`); it holds the projector's MAC and DHCP hostname.
6. Projector joins the hotspot.

## Capture

- 30 min idle on the launcher, then 30 min of use (settings, vendor apps, an app store if any).
- Destination list, redacted before pasting into the issue:

   ```bash
   tshark -r egress.pcapng -T fields -e ip.dst -e ipv6.dst -e dns.qry.name -e tls.handshake.extensions_server_name | sort -u | tools/redact.sh
   ```

- APK inventory: `pm list packages -f`, SHA-256 of each pulled APK, VirusTotal lookup **by hash only** (never upload the APK).

## Record

| Item | Result |
|---|---|
| Hotspot isolation pass/fail (Pi, router, tailnet unreachable) | |
| Destinations (domains) | |
| Unexpected or known-bad destinations | |
| Vendor packages and verdict per package | |

Afterwards (once runbook 05 has run if the review clause fired): forget the hotspot on the projector, Wi-Fi and Bluetooth off, and never configure another SSID.
