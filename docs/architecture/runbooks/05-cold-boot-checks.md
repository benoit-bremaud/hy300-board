# Runbook 05 — cold-boot checks (strategy A, or ADR-0003 review clause)

When: strategy A, or runbook 04 lowered C (ADR-0003 review clause). Runs right after runbook 03 on the **same isolated hotspot**, before it is forgotten.

## Steps

1. Enable network ADB by the method found in runbook 01 (`adb tcpip 5555` over USB, a firmware toggle, or `adb pair`).
2. Power off by IR, unplug mains 1 min, replug, power on; do not connect USB.
3. From the PC on the hotspot: `adb connect <projector-hotspot-address>:5555` → pass if it connects within 60 s.
4. If it fails: install the schaumann-byte keep-alive helper (or equivalent) and repeat step 2–3 once.
5. `KEYCODE_WAKEUP` from standby: `adb shell input keyevent KEYCODE_WAKEUP` with the projector in its "off" state → pass if the panel lights.

## Pass/fail to record

| Check | Result |
|---|---|
| ADB 5555 alive after a cold boot without USB | |
| Keep-alive helper needed | |
| `KEYCODE_WAKEUP` from standby | |

## Outcome

Feeds the §3 re-score of the plan: with runbook 03 clean and ADB 5555 alive, A rises to 375; together with a lowered C (350) the strategy flips and ADR-0003 is re-decided before confirmation. Then forget the hotspot (runbook 03, last step).
