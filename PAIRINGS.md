# WhatBox Pairings — master record

One entry per verified firmware + Android combination. The same `pair/<name>` tag exists in every relevant repo for each entry. Open this file first; supporting detail lives in `PROTOCOL.md`, `PID.md`, `FEATURES.md`, `SCOPING.md`, `GLOSSARY.md`.

All dates are +1000 (AEST/AEDT) unless noted. All SHAs are git object SHAs at the moment of writing (2026-04-20).

---

## pair/era2-classic — Verified, daily use

| Field | Value |
|---|---|
| **Firmware repo** | `whatbox-cnc-firmware-v4.1` |
| **Firmware ref** | tag `v4.2e` |
| **Firmware commit** | `786cf919ba84ddbe5dc7bf5107dbd529eda680ef` |
| **Firmware author** | Bradley King |
| **Android repo** | `whatbox-android-remote-control` |
| **Android ref** | branch `preserve/Gcode` |
| **Android commit** | `0431fc14d0b3c309610e335bc629ffdc548bfe8e` (2023-02-22) |
| **Android author** | Sven Tiede |
| **Android versionName** | `0.0.13-b1` |
| **Transport** | Bluetooth Classic SPP, UUID `00001101-0000-1000-8000-00805F9B34FB`, 115200 baud |
| **MCU** | Atmel ATSAMS70Q20 (ARM Cortex-M7, 1 MB flash, 384 KB SRAM), 16 MHz xtal |
| **Packet header** | `0x44` ('D') |
| **Packet types — firmware side** | 17 ASCII (M S B A P R T E L U W + V Z K I N + F/C) |
| **Packet types — Android side** | 23 ASCII (M S B A P R F T E L U C K Z V I W X G J H Q N) — superset |
| **Error correction** | 1 error byte per 7 data bytes, bit-7 always set (escapes 0x44 in payload) |
| **CRC** | CRC16 only on outgoing `W` and `C` packets |
| **EEPROM** | External AT24CXX via I²C |
| **Address map** | 0–5000 (encoder 1000–1299, motor 1050–1237, references 1300+, tuning 2050–2060, calibration 4000–5000) |
| **PID** | Inline position-only; gains Kp={2, 2, 0.9} Ki={1, 1, 0.5} Kd={0, 0, 0}; integral clamped; no feed-forward |
| **Verified** | 2024 on Bridgeport, full feature set |
| **Recommended use** | Day-to-day WhatBox operation |

---

## pair/era3-stairmaster-fluidnc — Verified, Sean's current kit

| Field | Value |
|---|---|
| **Firmware repo** | NONE — runs **external FluidNC** (open-source, GRBL-based) |
| **Firmware target** | ESP32-WROOM-32 on `daughter_board_v3` (via `whatbox-hardware`) |
| **FluidNC configs** | `cncEST.yaml` and `CamelCNC.yaml`, shipped inside the Android app's assets |
| **Config boards** | FluidNC `6x` and `6 Pack` ESP32 boards |
| **Android repo** | `whatbox-android-remote-control` |
| **Android ref** | branch `StairMaster` |
| **Android commit** | `1d81f86389b3922d84d95f687686181de14f8231` (2025-08-06) |
| **Android versionName** | `24.5.1-StairMaster` |
| **Android versionCode** | `300240501` |
| **Transport (primary)** | FluidNC GCode stream over Bluetooth Classic |
| **Transport (fallback)** | BT Classic SPP via legacy BlConnectService |
| **Packet types — Android side** | 23 ASCII (legacy set still present for backward compat) |
| **PID** | FluidNC built-in — GRBL-derived step generator, not a servo PID |
| **Step/motion engine** | I2S_STREAM, 4 µs pulse, 1 µs dir delay |
| **Drip-feed fix** | Android commit 2023-08-24 "Fixed dripfeeding to no longer overload Fluidnc" |
| **CNCEST connect** | Android commit 2024-03-06 "Made the samsung tablet connect to CNCEST" |
| **Verified** | 2025-08 on Sean's current machine |
| **Recommended use** | Current production pair |

---

## pair/mc-refactor-candidate — NOT VERIFIED, do not ship

| Field | Value |
|---|---|
| **Firmware repo** | `whatbox-cnc-firmware-v4.1` |
| **Firmware ref** | branch `origin/mc-refactor` |
| **Firmware commit** | `7ed6058` (HEAD, 2020-12-16) |
| **Firmware author** | Shravan Lal `<Shravan Lal>` |
| **Firmware branch size** | 40 commits exclusive to master, 99 total across branches, 2020-12-10 → 2020-12-16 |
| **Android repo** | `whatbox-android-remote-control` |
| **Android ref** | branch `preserve/bluetooth` |
| **Android commit** | `947c11f141793e12d4fcbe505dad72c0452139cb` (2023-09-13) |
| **Android versionName** | `0.0.13-b1` |
| **Transport** | Bluetooth Classic SPP, 115200 baud |
| **Packet types — firmware side** | 17 ASCII (v4.2e baseline — refactor did not change wire protocol) |
| **Packet types — Android side** | 23 ASCII (full superset) |
| **PID** | **Shravan Lal's cascaded dual-loop.** Position loop (slow) drives velocity reference; velocity loop (fast) drives motor duty. Per-axis feedforward coefficient. First-tick derivative suppression. Time-scaled integration (`integrator += error · dt`). |
| **Refactor module tree** | `src/refactor/{pid,follower,positioner,movement,preloader,math_util,parser,generic_cmd,protocol}.{c,h}` |
| **follower.h size** | 0 bytes (empty blob `e69de29`) — WIP marker |
| **Last on-hardware test** | 2020-10-21 ("WIP 20201021 — Tested on motors with PID and accel/decel ramps") — nothing since |
| **WIP commit tombstones** | `7d28ac3` "Unfinished implementation of cascaded PID control for 'follower'"; `8e16931` "wip 20201210"; `f604db9` "wip trash"; `771bb9e` "untested integration into main.c"; `cdd1e03` "Hacked refactored code into main loop" |
| **Verified** | **NO** |
| **Gap analysis** | see `SCOPING.md` |
| **Effort to finish** | **18–27 engineer-days** low-to-realistic range, 35+ pessimistic |
| **Top 3 risks** | (1) PID follower never wired to motors — 2–3 day code + 3–5 day scope tuning; (2) homing + probing entirely absent — 4–6 days combined, safety-critical; (3) EEPROM + comms layer missing — 4–5 days, blocks all end-to-end testing |
| **Integration hazard already present** | `main.c` calls both legacy `positionController()` AND refactor `positioner__doMovementTick()` on every loop iteration — dual code paths; silent divergence risk once refactor wiring is completed |
| **Recommended use** | Reference only — study the PID architecture; port the cascade into a blessed branch after the ~20 engineer-days of completion work |

---

## How to add a new pairing

1. Verify firmware + Android work together on real hardware. Note the two commit SHAs.
2. Pick a short kebab-case name: `pair/<year>-<feature>` or similar (e.g. `pair/2026-spring-probe-fix`).
3. In the firmware repo: `git tag pair/<name> <firmware-sha>` (add `-a -m "…"` for an annotated tag with notes).
4. In the Android repo: `git tag pair/<name> <android-sha>`.
5. (Optional) Push tags: `git push origin pair/<name>` in each repo.
6. Add a full row to this file with the same field list as above. Commit and push here.

Rule: same tag name = compatible pair. Different names = don't mix. Forking a variation = fork the name (`pair/era2-classic` → `pair/era2-with-probe-fix`).

See `GLOSSARY.md` for what Era 1/2/3 mean and the Shravan / Bradley `<Shravan Lal>` identity distinction.
