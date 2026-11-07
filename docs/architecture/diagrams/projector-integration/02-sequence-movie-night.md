# Sequence diagram — projector-integration — movie night

> **Feature**: epic #1 — integrate the HY300 projector into Home Assistant and the homelab
> **Source specs**: `docs/architecture/specs/projector-integration-requirements.md` REQ-1, REQ-2, REQ-3, NFR-3
> **Related ADRs**: ADR-0003, ADR-0004, ADR-0005
> **Realises**: UC1 · **See also**: `03-component.md`, `05-state-projector.md`

## Context

The one-tap scene from a cold setup to PopCornTV on screen. Film selection (UC2) is out of scope: it happens on the box with the remote. HA has no feedback channel from the projector: the only optional feedback is a smart plug's power reading.

## Diagram

```mermaid
sequenceDiagram
  title sd — UC1 Start a movie night
  actor User as FamilyMember / Owner
  participant HA as Home Assistant
  participant IR as IR blaster
  participant Proj as Projector
  participant MediaBox as Box
  participant PCTV as PopCornTV (behind Caddy)

  User->>HA: tap "Movie night"
  HA->>HA: lights scene
  HA->>IR: send POWER code
  IR-->>Proj: IR POWER
  Proj->>Proj: boot (Booting → VendorLauncher or HdmiInput)
  opt smart plug with power reading present
    HA->>HA: wait for draw rise (timeout = boot time of runbook 04 + margin)
    alt no draw rise
      HA->>IR: retry POWER once
    end
  end
  HA->>IR: send INPUT HDMI code (delay = boot time of runbook 04 + margin, skipped if the input is remembered)
  IR-->>Proj: IR INPUT
  HA->>MediaBox: power on (integration over the tailnet, host set by hand)
  MediaBox-->>HA: state on (NFR-3: ≤ 60 s)
  HA->>MediaBox: launch PopCornTV
  MediaBox->>PCTV: HTTPS GET (tailnet, CA trusted or ACME)
  PCTV-->>MediaBox: web player
  MediaBox-->>Proj: HDMI video + audio
  HA-->>User: notification "ready" (or "projector state unknown" without a plug reading)
```

## Notes

- POWER is a toggle: a blind retry could switch a lit projector off, so the retry exists only with the plug reading (UC1 extension 2a).
- The box's integration ports and host are fixed by hand (ADR-0004: no mDNS over the tailnet).
- Volume is not part of the scene; it is set on the box's `media_player`.
