# ADR-0004 — Put the media box on the guest SSID and the tailnet

Status: accepted (box variant `guest-ts` validated by the owner on 2026-11-06; the projector variant of D3 was decided under strategy A, the box matrix was added in review round 4) · Date: 2026-11-06 · Decision D3 (box variant), plan §4 · Last changed: owner validation of the box variant, after pre-push review round 1

## Context

From the LAN, PopCornTV is unreachable by design (Caddy bound to loopback and the tailnet IP, bb-homelab ADR 0007). The box runs vendor firmware too, even if trusted more than the projector's. On the main Wi-Fi it would see SSH 22, Mosquitto 1883, the operator router's UI and every device.

## Decision drivers (weights /100)

Need 35 · security 30 · simplicity 20 · coherence with bb-homelab patterns 15.

## Options considered (box)

| Option | Need | Security | Simplicity | Coherence | Total /500 |
|---|---|---|---|---|---|
| **guest-ts: guest SSID + Tailscale tagged node + default-deny ACL + Caddy 403** | 5 : all reachable | 4 : vendor firmware isolated from the LAN, ACL on the tailnet | 3 : SSID, Tailscale, ACL, DNS, CA | 5 : same pattern as family access | **430** |
| main-ts: main Wi-Fi + Tailscale | 5 : all reachable | 3 : box sees SSH 22, 1883, every device | 4 : no second SSID | 4 : Tailscale pattern, no isolation | 405 |

Sensitivity: main Wi-Fi passes guest only below security ≈ 7. The projector-variant matrix (main-wifi-caddy 245, guest-ts 460, guest-only 340, main-ts 345) is in plan §4 and applies only under strategy A.

## Outcome

Guest SSID; Tailscale as a tagged node (`tag:media-box`, pre-auth key, key expiry off); tailnet ACL default-deny (box → Pi on 443 and the integration's ports from the Pi to the box); Caddy `remote_ip` matcher returning 403 to the box on every host but PopCornTV; name resolution through a resolver on the Pi (split DNS) or the public domain of bb-homelab #28 with ACME; throughput gate: direct path and ≥ 2× the highest direct-play bitrate.

## Consequences

- Three bb-homelab bricks block v0.2: ACL policy, Caddy matcher, name resolution.
- mDNS discovery does not cross the tailnet: the HA integration gets the box host by hand.
- The box's player must trust the Caddy root or #28 must land first (ADR-0003 consequence).
