# Decision guide — "I want to build X, which pair do I use?"

Worked examples for picking a firmware + Android + transport + hardware combination. **Any AI working on WhatBox should walk through this before recommending a version pairing.**

If the scenario you care about is not listed, use `pairings.json` (machine-readable) as the index and add a new scenario here after you've verified it on hardware.

---

## Scenario 1 — "I want to make a Stairmaster that runs on Bluetooth"

**Answer:** `pair/era3-stairmaster-fluidnc`.

| Layer | Use |
|---|---|
| Android app | `whatbox-android-remote-control` · branch `StairMaster` · v24.5.1-StairMaster |
| Firmware | **external FluidNC** on an ESP32 — NOT in any whatbox repo. Loaded onto the ESP32 with the `cncEST.yaml` or `CamelCNC.yaml` config that ships inside the Android app's assets folder. |
| Bluetooth | **Classic Bluetooth SPP** (UUID `00001101-0000-1000-8000-00805F9B34FB`). BLE MLDP is also supported as a fallback but classic SPP is the primary for Stairmaster. |
| MCU | ESP32-WROOM-32 (+ SAM E70 co-processor on the daughter board) |
| PCB | `whatbox-hardware` · `daughter_board_v3` (the current board, supercap UPS + cellular modem, Sean's 2026-01-12 edits) |
| Transport protocol | GCode line stream (GRBL-compatible). Status polled with the `?` character every 250 ms. |
| Verified | 2025-08, Sean's current production kit |
| SourceTree tags | `pair/era3-stairmaster-fluidnc` on the Android repo only (no firmware tag — FluidNC is external) |

**Why NOT the whatbox firmware v4.1?** The Stairmaster Android build (`StairMaster` branch) was designed to drive FluidNC's GCode stream. It still carries the legacy 23-type packet set for backward compatibility, but its motion pipeline expects FluidNC. Pairing it with whatbox-firmware-v4.1 would mostly work for status queries and jog, but spindle/tool/dripfeed commands rely on FluidNC-specific behaviour.

---

## Scenario 2 — "I want to run my existing Bridgeport mill with the classic WhatBox firmware"

**Answer:** `pair/era2-classic`.

| Layer | Use |
|---|---|
| Android app | branch `preserve/Gcode` (v0.0.13-b1, 2023-02-22) — the 23-type full-superset build |
| Firmware | `whatbox-cnc-firmware-v4.1` · tag `v4.2e` (latest formal release, 17-type set, inline position-only PID) |
| Bluetooth | Classic Bluetooth SPP via Microchip RN4020 module · 115200 baud UART |
| MCU | ATSAMS70Q20 (ARM Cortex-M7 on daughterBoard v2) |
| PCB | `whatbox-hardware` Controller_v2f + daughterBoard v2 (ELECROW 2017-01-14 order, still current if unchanged) |
| Transport protocol | WhatBox binary packets, header `0x44`, packet types A–Z, CRC16 on `W` and `C` only |
| Verified | 2024 on Bridgeport, full feature set |
| SourceTree tags | `pair/era2-classic` in both firmware and Android repos |

**Do not use firmware tag `v4.1d`.** It silently resized the `S` status packet. Android streaming parsers will desynchronise. Use `v4.2e` (grew by adding types, not by resizing) or `V4.1b` / `v4.1c` (baseline, no break).

---

## Scenario 3 — "I want to study or finish Shravan's gold-PID refactor"

**Answer:** `pair/mc-refactor-candidate` — reference only, not production.

| Layer | Use |
|---|---|
| Android app | branch `preserve/bluetooth` (v0.0.13-b1, 2023-09-13) — 23-type superset |
| Firmware | `whatbox-cnc-firmware-v4.1` · branch `origin/mc-refactor` at SHA `7ed6058` (2020-12-16) |
| Bluetooth | Classic Bluetooth SPP, 115200 baud (same transport as era 2) |
| PID | Shravan Lal's cascaded dual-loop in `src/refactor/pid.c` + `follower.c`. Per-axis feed-forward, first-tick derivative suppression, dt-scaled integrator. |
| Status | **NOT VERIFIED** — the PID output is never wired to the motors in main.c. Homing, probing, EEPROM persistence, and the comms layer are all absent. |
| Effort to ship | **18–27 engineer-days** (see `project_mc_refactor_scoping.md` in Claude memory) |
| Verified | Only 2020-10-21 bench test — nothing since |
| SourceTree tags | `pair/mc-refactor-candidate` in both firmware and Android repos |

---

## Scenario 4 — "I want to add Telnet/WiFi control to the classic v4.1 firmware"

**Answer:** Use the Android `preserve/4.1-modernization` branch + `whatbox-espbluetooth` (ESP32 bridge) + v4.1 firmware.

| Layer | Use |
|---|---|
| Android app | branch `preserve/4.1-modernization` (2026-01-12) — the ONLY Android build with a Telnet driver |
| Firmware | `whatbox-cnc-firmware-v4.1` master or tag `v4.2e` |
| Bridge | `whatbox-espbluetooth` on an ESP32 — bridges Telnet/TCP port 23 to the SAM E70's UART |
| Bluetooth | BT Classic SPP is still available as a fallback; BLE was scaffolded but Telnet is the new primary |
| Status | In-progress. Not independently verified end-to-end as of 2026-04-20. |
| SourceTree tags | No pair tag issued yet — it's WIP |

---

## Scenario 5 — "I'm looking at an old v2.0 firmware build and want to know which Android was used"

**Answer:** Android tag `1.0b` (2016-01-15).

| Layer | Use |
|---|---|
| Android app | tag `1.0b` on `whatbox-android-remote-control` — the 2016 beta |
| Firmware | `whatbox-cnc-firmware-v2.0` (xmega128 repo) |
| Bluetooth | Classic Bluetooth SPP via RN4020 · **38400 baud** (not 115200) |
| Packet encoding | **Numeric** packet types (1, 2, 4, 8) — NOT ASCII A–Z |
| Status | Archival only. Era 1 hardware (xmega128A1U + original motherboard). |

**Warning:** v2 and v4 firmware both start packets with `0x44`. They are NOT interchangeable — baud, packet encoding, and error correction all differ. Never pair an Era-1 Android build with Era-2 firmware or vice versa.

---

## How to add a new scenario

1. Test a firmware + Android + transport combination on real hardware until you are confident it works.
2. Pick a short pair name: `pair/<year>-<feature>` (e.g. `pair/2026-spring-probe-fix`).
3. In the firmware repo: `git tag pair/<name> <firmware-sha>` — push with `git push origin pair/<name>` if you want it shared.
4. In the Android repo: `git tag pair/<name> <android-sha>` — same push convention.
5. Add an entry to `pairings.json` with the full field set.
6. Add an entry to `PAIRINGS.md` with the human-readable row.
7. Add a new scenario section to this file describing what the pair is for.
8. Commit and push this repo.

Rule: same `pair/*` tag name across repos = compatible combination. Different names = don't mix.

---

## For AI agents — read this before making recommendations

- `pairings.json` is the authoritative machine-readable index. Parse it, do not reinvent the data.
- `PAIRINGS.md` is the human-readable companion.
- `PROTOCOL.md` documents the three wire-protocol eras (v2, v4, FluidNC). The `0x44` header is shared across eras — **do not** treat a matching header as evidence of compatibility.
- `PID.md` explains why `origin/mc-refactor` is the gold-PID reference but not shippable.
- When Sean says "Shravan" or "Siobhan" (dictation variant), he means the real developer Shravan Lal `<Shravan Lal>` whose work lives on `origin/mc-refactor`. Do NOT confuse with the `origin/shravan` branch (which is Denis Raison's 2019 handoff) or with Bradley King's `<Shravan Lal>`-email identity-bug commits on master.
- The `pair/*` tags are the primary artifact. If a tag exists in both the firmware and Android repos with the same name, those SHAs are declared compatible. If not, treat them as incompatible until proven otherwise.
