# pair/era3-stairmaster-fluidnc · VERIFIED · Current production (Stairmaster)

Firmware: external FluidNC on ESP32-WROOM-32 (NOT in any whatbox repo)
Android:  whatbox-android-remote-control branch StairMaster (SHA 1d81f863) — v24.5.1-StairMaster
Transport: Bluetooth Classic + FluidNC GCode stream (line-based, ? polling every 250ms)
Configs: cncEST.yaml + CamelCNC.yaml (in Android app assets)
PID: FluidNC built-in (GRBL step generator)
Hardware: daughter_board_v3 (ESP32 + SAM E70 + supercap UPS + cellular modem)
Verified: 2025-08 — Sean's current kit
