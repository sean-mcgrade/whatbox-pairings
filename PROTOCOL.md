# WhatBox Wire Protocol — per-era fingerprint

The WhatBox ecosystem has passed through three protocol generations. All three use the same **header byte 0x44** (`'D'`), which is how they get confused. Everything else differs.

---

## Era 1 — XMEGA / Classic BT SPP (2016-05 → 2017-01)

| Field | Value |
|---|---|
| Firmware repo | `whatbox-cnc-firmware-v2.0` |
| MCU | Atmel ATxmega128A1U (8-bit AVR), 16 MHz |
| Baud | **38400** |
| Transport | UART over Microchip RN4020 classic Bluetooth SPP |
| Packet header | `0x44` |
| Packet type encoding | **numeric** (1 = DATA, 2 = CONFIG, 4 = STATUS, 8 = REFERENCE) |
| Packet layout | `[0x44][type byte][42-byte payload][6 error bytes]` |
| Error correction | 1 error byte per 7 data bytes (bit-7 always set, escapes 0x44 in payload) |
| CRC | none |
| EEPROM | Internal xmega EEPROM |
| Command model | Address-based, 0x0000–0x001A+ |
| Android app tag | `1.0b` on `whatbox-android-remote-control` (2016-01-15) |

## Era 2 — SAM E70 / "Bluetooth 4" protocol (2017-01 → 2022-02)

| Field | Value |
|---|---|
| Firmware repos | `whatbox-cnc-firmware-v4.0` (Jan–Feb 2017 port), `whatbox-cnc-firmware-v4.1` (current) |
| MCU | Atmel ATSAME70Q20 / ATSAMS70Q20 (ARM Cortex-M7, 1 MB flash, 384 KB SRAM), 16 MHz xtal |
| Baud | **115200** (upgrade from Era 1's 38400) |
| Transport | UART over RN4020 classic SPP (still); "BT4" refers to protocol v4, not Bluetooth 4.0 |
| Packet header | `0x44` |
| Packet type encoding | **ASCII A–Z** (NOT numeric) |
| Packet types (v4.0) | M, S, B, A, P, R, T, E, L, U (10) |
| Packet types (v4.1c / V4.1b baseline) | A, B, E, L, M, P, R, S, T, U, W (11 — adds W for query) |
| Packet types (v4.1d) | previous + V, Z (**BREAKING: S payload size grew**) |
| Packet types (v4.2e) | previous + V, Z, K, I, N (17 total) |
| Packet types (Gcode / Stairmaster(lowercase) / v2-alpha / build-output-stairmaster) | adds H, J, Q (20 total; Gcode adds X = 21) |
| Packet types (brad-bridgePortTesting / devOld) | adds F stub (18 total) |
| Packet layout | `[0x44 'D'][type char A–Z][payload ≤119 B][1 err per 7 data bytes]` |
| Error correction | 1 error byte per 7 data bytes, bit-7 always set |
| CRC | CRC16 only on outgoing `W` (whatsup) and `C` (command) packets |
| EEPROM | External AT24CXX via I²C (SAM has no internal EEPROM) |
| EEPROM PID addresses | X Kp@0x4F/Ki@0x53/Kd@0x57; Y @0x81/0x85/0x89; Z @0xB3/0xB7/0xBB; axis stride 50 bytes |
| Address map | 0x0000–0x13A8 (0–5032), expanded to 0–6000 in Gcode family |
| Android app tag range | from `1.0b` up to tag `LastSafeBridgeportVersion` (2022-02-23) |

### Packet character glossary (Era 2)

| Char | Meaning | Size (bytes) |
|---|---|---|
| M | MAIN (XYZ pos, worklist) | 16 incoming |
| S | STATUS (e-stop, mode, locks, boundaries) | 11 incoming (v4.2e: resized) |
| B | BOUNDARIES | 18 |
| A | ABSOLUTE position request | 9 |
| P | PROBING data | 9 |
| R | REFERENCE points (packed) | 30 |
| F | SINGULAR_REFERENCE (single ref) | 10 |
| T | TUNING (PID + motor gains) | 39 / 63 |
| E | ENCODER config | 21 / 15 |
| L | LIMITS (full matrix) | 54 / 24 |
| U | BUILDVERSION (10B date + 30B hash) — runtime firmware version detection | 40 |
| C | COMMANDS (outgoing, +CRC16) | 6 |
| V | VOLUME / dimensions | 18 |
| W | WHATSUP / ping (outgoing, +CRC16) | 1 |
| X | DEBUG_STRING (Gcode only) | 40 |
| K | EEPROM_CHECK (integrity) | 8 |
| Z | PID_CONTROL matrix | 66 |
| G | LOGGER output | 16 |
| I | SPINDLE status | 7 |
| N | SPEED (feed rate / spindle) | 15 |
| J | PRELOADER (file upload buffer, Gcode) | 15 |
| H | POSITIONER (coord transform, Gcode) | 24 |
| Q | MISC | 12 |
| O | EEPROM_DATA (Android `4.1-modernization` only — no firmware counterpart in v4.1; targets a future firmware or an EspBlueTooth bridge) | ? |

### The v4.1 branch landscape — what each branch changes on the wire

| Branch | Packet set | Baud | Addresses | Notes |
|---|---|---|---|---|
| tag `V4.1b` / `v4.1c` | 8 (A B E L M P R S T U W) | 115200 | 0–99, 999, 2050–2060 | "All core features embedded" — baseline stable |
| tag `v4.1d` | 10 (+V, +Z, S resized) | 115200 | +2050–2060 | **Status packet grew — breaks fixed-size parsers** |
| tag `v4.2e` | 17 (+V, +Z, +K, +I, +N) | 115200 | 0–5000 | Latest formal release |
| `origin/master` (HEAD) | 17 (v4.2e set) | 115200 | 0–5000 | Tainted by Bradley `<Shravan Lal>` identity-bug commits (not Shravan's PID work) |
| `origin/mc-refactor` | 17 (v4.2e set) | 115200 | 0–5000 | **Shravan Lal's real branch; refactor is internal, wire protocol unchanged** |
| `origin/shravan`, `test_SPI`, `test_SPI2` | 17 | 115200 | 0–5000 | Three labels, one Denis Raison 2019 commit `8c6a493f`; NOT Shravan's code |
| `origin/dev`, `feature/EXTRA_REF`, `feature/maxon_motion_control` | 17 | 115200 | 0–5000 | master-family protocol |
| `origin/Gcode` | 21 (+H, +J, +Q, +X) | 115200 | 0–6000 | Full GCode packet subset |
| `origin/Stairmaster` (lowercase), `v2-alpha`, `build-output-stairmaster` | 20 (+H, +J, +Q) | 115200 | 0–6000 | GCode subset without debug channel |
| `origin/brad-bridgePortTesting`, `devOld` | 18 (+F stub) | 115200 | 0–5000 | F has empty handler |
| `origin/hotFix`, `main-cleanup`, `referencePointsFix`, `lookahead` | 17 | 115200 | 0–5000 | Same wire; hotFix strips tuning UI |
| `origin/I2C_DIODE_2020`, `Implement_I2C`, `test_SPI`, `vescTesting` | 17 | 115200 | 0–5000 | Hardware bring-up only |

---

## Era 3 — FluidNC / ESP32 (2022-02 → present)

| Field | Value |
|---|---|
| Firmware | **External FluidNC** (open-source, GRBL fork); NOT in any whatbox repo |
| Bridge hardware | ESP32-WROOM-32 + SAM E70 co-processor on `daughter_board_v3` |
| Transport | **BLE** using UBLOX / Microchip MLDP proprietary profile (service UUID `2456e1b9-26e2-8f83-e744-f34f01e9d701`, data char UUID `2456e1b9-26e2-8f83-e744-f34f01e9d703`, CCCD `00002902-0000-1000-8000-00805f9b34fb`) |
| Fallback transport | Classic BT SPP (UUID `00001101-0000-1000-8000-00805F9B34FB`) — retained |
| Protocol | **GCode** (GRBL-compatible); YAML config upload via Xmodem |
| Status poll | `?` character every 250 ms (FluidNC idiom) |
| Config files | `cncEST.yaml`, `CamelCNC.yaml` (shipped inside Android app assets; board `6x` or `6 Pack`) |
| Cutover marker | Android tag `LastSafeBridgeportVersion` (2022-02-23) — last build fully compatible with Era 2 binary protocol |
| Android app | `StairMaster` branch, versionName `24.5.1-StairMaster`, versionCode `300240501` |

---

## Cross-era compatibility gotcha

Firmware `v2.0` and `v4.x` **both use header byte `0x44`**. On a logic analyser the framing looks identical. But:

- Baud differs (38400 vs 115200) — wrong baud = no sync
- Packet-type encoding differs (numeric 1/2/4/8 vs ASCII A–Z) — wrong encoding = garbage parse
- v4.0 had 10 types, v4.1d added V+Z AND resized S, v4.2e added +K/I/N without resizing, Gcode-family added +H/J/Q/X

**Never pair an Era-1 Android build (`1.0b` tag) with any Era-2 firmware.** Never pair `v4.1d` with any Android — the S resize silently desynchronises the stream. Use `v4.1c` / `V4.1b` (baseline) or `v4.2e` (grew by adding types, not by resizing).

---

## Runtime firmware-version detection

Android uses the 'U' BUILDVERSION packet to read firmware identity at runtime:

```
byte 0-9   : build date (10 chars)
byte 10-39 : firmware hash / version string (30 chars)
```

Stored in SharedPreferences as `build_version` and `build_date`. Shown in `UserSettingActivity.java` → "Firmware Build Version" / "Firmware Build Date". 'U' exists in v4.0 onwards; does NOT exist in v2.0.
