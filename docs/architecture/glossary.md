# Glossary

Every domain term used in the diagrams and specs, defined once. Alphabetical, English headword.

| Term | Definition | Source |
|---|---|---|
| ACL | Tailscale access-control policy of the tailnet; default-deny for `tag:media-box`. | ADR-0004 |
| ADB | Android Debug Bridge: USB or TCP (port 5555) control channel of an Android device; the channel strategy A would use. | AOSP |
| Booting | Projector state between power-on and the first UI (vendor launcher or HDMI input). | `05-state-projector.md` |
| Box | The trusted HDMI box (`Box` in diagrams, `media box` in ADR titles and the tailnet tag): a device chosen in v0.2 that plays PopCornTV and is driven by Home Assistant. | plan §5 v0.2 |
| Caddy | Reverse proxy of bb-homelab serving PopCornTV over HTTPS with an internal CA (bb-homelab ADR 0007); returns 403 to the box on every host but PopCornTV (this repo's ADR-0004). | bb-homelab, ADR-0004 |
| Dump | Read-only image of the projector's eMMC, verified by two identical reads (whole image in Loader mode, read-only partitions on a live read), stored encrypted, shared as hashes only. | runbook 02 |
| Egress capture | Packet capture of the projector's first network contact on an isolated hotspot, listing every destination. | runbook 03 |
| Family member | Anyone using the projector with the remote (actor `FamilyMember`): the owner's daughters, guests. | plan §1 |
| Full off | Projector state `Off`: mains kept, panel dark, standby draw measured; distinct from `Unplugged`. | runbook 04 |
| Guest SSID | Isolated Wi-Fi network of the operator router that the box joins; no access to the LAN. | ADR-0004 |
| HA | Home Assistant, running on the homelab Pi in host network. | bb-homelab |
| HdmiInput | Projector state showing the HDMI input (the box's picture). | `05-state-projector.md` |
| HDMI input | The projector's only video input; the RK621 bridge feeds the panel from it. Selected by IR. | board identification |
| Homelab Pi | The Raspberry Pi 5 of bb-homelab hosting HA, Caddy, PopCornTV and a Tailscale node. | bb-homelab |
| IR blaster | Infrared transmitter (ESP-based or Broadlink) driven by HA through a `remote` entity, replaying the remote's codes to the projector. | ADR-0005 |
| IR receiver | Capture device (ESPHome `remote_receiver`, Broadlink learn mode, Flipper) used once to record the remote's codes. | runbook 04 |
| Loader mode | Rockchip USB mode entered by `RST/RECOVERY` + USB or `adb reboot loader`, allowing `rkdeveloptool rl` reads with the OS not running. Conditional on a Rockchip SoC. | board identification |
| Maintainer | The person running the runbooks on the hardware (actor `Maintainer`): the owner or a third party. | plan §6 |
| Maskrom | Rockchip BootROM USB mode (eMMC CLK/D0 short at power-on), needs a matching `rkbin` loader loaded into RAM; last-resort dump path. | runbook 02 |
| media_player | HA entity exposed by the box integration: state, power, volume, app launch. | `03-component.md` |
| Movie night | The HA scene: lights, projector on by IR, HDMI selected, box on, PopCornTV launched. | UC1 |
| Operator router | The 5G operator box of the home (CGNAT, no port forwarding) providing the main Wi-Fi and the guest SSID. | plan §2 |
| OTG | USB On-The-Go: the USB-A port acting as a device port so the PC can reach ADB / Loader / Maskrom. | runbook 01 |
| Owner | The maintainer of bb-homelab and PopCornTV who validates decisions (actor `Owner`). | plan |
| PopCornTV | The owner's self-hosted media server and web player, HTTPS only (popcorntv ADR-0010). | popcorntv |
| Projector | The HY300-branded unit under study (`Projector` in diagrams): display plus IR receiver, never on a network after recon. | ADR-0003 |
| Redaction | Removal of serial numbers, MACs, SSIDs/PSKs, Tailscale keys, IPs and EXIF from every artefact before pasting or staging. | NFR-2 |
| remote entity | HA entity of the IR blaster integration sending named codes. | `03-component.md` |
| RK621 | Rockchip HDMI-to-MIPI/LVDS bridge on the mainboard; the reason HDMI is an input. | board identification |
| Scheduler | Non-human initiator of UC3 (sleep timer, a "later" feature). | `01-use-case.md` |
| Smart plug | Mains plug with power reading; optional feedback on the projector's state, fallback power-on path of ADR-0005. | ADR-0005 |
| Strategy A / B / C | Stock Android + ADB / Linux on the board / trusted HDMI box (adopted). Sub-decision options use words (`ir`, `plug`, `manual`; `guest-ts`, `main-ts`), never these letters. | ADR-0003 |
| Tailnet | The owner's Tailscale network; the box joins it as a tagged node under a default-deny ACL. | ADR-0004 |
| Tailscale | Mesh VPN used by bb-homelab; its client runs on the box and its node on the Pi. | bb-homelab |
| Unplugged | Projector state with mains removed. | `05-state-projector.md` |
| VendorLauncher | Projector state showing the stock Android launcher after boot. | `05-state-projector.md` |
| Web player | The browser or app on the box that renders PopCornTV; chosen by the v0.3 MCDA. | D6 |
