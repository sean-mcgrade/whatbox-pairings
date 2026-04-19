# WhatBox Pairings

One entry per verified firmware + Android combination.
Same `pair/<name>` tag exists in both the firmware repo and the Android repo for each entry.

---

## pair/era2-classic  —  Verified, daily use

- **Firmware:** whatbox-cnc-firmware-v4.1 @ tag `v4.2e`
- **Android:** whatbox-android-remote-control @ branch `preserve/Gcode` (2023-02-22)
- **Transport:** Bluetooth Classic SPP, 115200 baud
- **PID:** master defaults (inline position-only, Kp={2,2,0.9} Ki={1,1,0.5})
- **Notes:** Recommended working pair. Full feature set.

---

## pair/era3-stairmaster-fluidnc  —  Verified, Sean's current kit

- **Firmware:** external FluidNC on ESP32 (not in any whatbox repo)
- **Android:** whatbox-android-remote-control @ branch `StairMaster` (2025-08-06)
- **Transport:** BT Classic + FluidNC GCode stream
- **PID:** FluidNC built-in (GRBL-derived)
- **Notes:** Current production pair as of 2025-08.

---

## pair/mc-refactor-candidate  —  NOT VERIFIED, do not ship

- **Firmware:** whatbox-cnc-firmware-v4.1 @ branch `origin/mc-refactor` (7ed6058)
- **Android:** whatbox-android-remote-control @ branch `preserve/bluetooth` (2023-09-13)
- **Transport:** Bluetooth Classic SPP, 115200 baud
- **PID:** Shravan Lal's cascaded dual-loop + feedforward (unfinished)
- **Notes:** Gold-standard PID architecture but PID never wired to motors. ~20 engineer-days to finish. See `project_mc_refactor_scoping` in Claude memory.

---

## How to add a new pairing

1. Verify firmware + Android work together on real hardware.
2. Pick a short pair name, e.g. `pair/2026-spring-test`.
3. In the firmware repo: `git tag pair/2026-spring-test <sha>`
4. In the Android repo: `git tag pair/2026-spring-test <sha>`
5. Add an entry to this file.
6. Commit here and push.

Same tag name = compatible pair. Different names = don't mix.
