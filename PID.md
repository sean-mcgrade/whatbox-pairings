# WhatBox PID — implementations by branch

Two distinct PID architectures live in the WhatBox firmware v4.1 repo: the **standard inline position loop** (used by master, every release tag, and most branches), and **Shravan Lal's cascaded refactor** (mc-refactor branch only, unfinished). This file is the reference for both.

---

## Standard PID — used by master, v4.2e, v4.1d, v4.1c, V4.1b, dev, hotFix, referencePointsFix, lookahead, feature branches, all hardware-test branches

**Structure:** single-loop, position-only PID per axis. No cascaded velocity loop. No feed-forward term.

**Location:** inline in `motion_controller.c` (approx. line 1850).

**Algorithm (master branch):**

```c
for (int i = 0; i < 3; i++) {
    axisVelocity[i]  = 1000 * positionError[i] * positionGain_P[i];
    axisVelocity[i] += positionIntegral[i] * positionGain_I[i];
    axisVelocity[i] += positionDerivative[i] * positionGain_D[i];
}
```

**Default gains:**

| Axis | Kp | Ki | Kd |
|---|---|---|---|
| X | 2.0 | 1.0 | 0.0 |
| Y | 2.0 | 1.0 | 0.0 |
| Z | 0.9 | 0.5 | 0.0 |

Kd is effectively unused across all three axes.

**Anti-windup:** integral clamped to `positionIntegralLimit` at every step.

**Derivative term:** derivative-on-error (not on measurement), no first-tick suppression, no filter — susceptible to noise spikes on setpoint change.

**Issues noted in audit:**
- Factor of 1000 on the proportional multiplier implies poorly tuned gain scaling
- No feed-forward → motion lag on high-feed-rate moves
- Monolithic, tightly coupled to `motion_controller.c` — hard to test in isolation

**hotFix branch specifically** strips the tuning UI (commit `f374f4f` "Removed all tuning code") — suggesting the gains are considered stable and not for user manipulation on that variant.

---

## Shravan Lal's cascaded refactor — mc-refactor branch only

**Structure:** dual-loop cascaded controller with per-axis feed-forward.

**Location:** dedicated module at `src/refactor/pid.c/h` plus `follower.c/h`, `positioner.c/h`, etc.

### Low-level PID module (`refactor/pid.c`)

```c
typedef struct pid__Instance {
    float Kp;
    float Ki;
    float Kd;
    float integrator;
    float lastError;
    bool  firstTick;
} pid__Instance;

float pid__doTick(pid__Instance *instance, float error, float dt) {
    instance->integrator = instance->integrator + error * dt;
    float difference = (error - instance->lastError) / dt;

    if (instance->firstTick) {
        instance->firstTick = false;
        difference = 0;   // suppress derivative spike on first tick
    }

    float result = (error * instance->Kp)
                 + (instance->integrator * instance->Ki)
                 + (difference * instance->Kd);
    instance->lastError = error;
    return result;
}
```

**Features of the low-level module:**
- Reusable across axes and loops (a single `pid__Instance` per usage)
- **First-tick derivative suppression** — eliminates the startup spike that the master PID suffers from
- **Time-scaled integration** (`integrator += error * dt`) — robust to variable sample rate
- **Derivative-on-error** with filter-by-dt — noise handling is better than master's raw subtraction
- No explicit anti-windup clamp — relies on stable gain tuning; could be a weakness

### Cascaded follower (`refactor/follower.c`)

```c
float follower__doVelocityTick(pid__Instance *instance,
                               float feedforwardCoefficient,
                               float reference, float velocity,
                               float dt) {
    float error              = reference - velocity;
    float feedforwardDuty    = reference * feedforwardCoefficient;
    float dutyCompensation   = pid__doTick(instance, error, dt);
    return feedforwardDuty + dutyCompensation;
}
```

**Architecture:**
- **Outer (position) loop** runs at Δt ≈ 1 ms — generates velocity reference from position error
- **Inner (velocity) loop** runs at Δt ≈ 0.1 ms — drives motor duty from velocity error
- **Feed-forward** — reference velocity multiplied by a per-axis coefficient, added straight to the duty output; PID only corrects the residual error

**Why this is better than master's inline PID:**
- Cascaded control is the textbook approach for servo motion — decouples position tracking from velocity regulation
- Feed-forward removes lag on known-velocity moves; PID only handles disturbance
- First-tick suppression eliminates startup jerk
- dt-scaled integrator survives changes in loop rate without gain retuning
- Modular — testable in isolation, replaceable, re-usable for additional axes

**Why it's not ready to ship:**
- Shravan's own commit `7d28ac3`: *"Unfinished implementation of cascaded PID control for 'follower'"*
- `follower.h` is **empty (0 bytes)** — the declarations that callers would need are missing
- `follower__doVelocityTick()` return value is **never assigned to motor drivers** — the duty output goes nowhere
- `follower__doTick()` is declared but never invoked from the main loop
- Last on-hardware test evidence is commit `4afbb99` 2020-10-21 *"WIP 20201021 — Tested on motors with PID and accel/decel ramps"* — everything after that is `wip trash`, `wip 20201210`, `Hacked refactored code into main loop`
- Parallel dual code paths in `main.c` — legacy `positionController()` and refactor `positioner__doMovementTick()` both run every iteration

---

## EEPROM PID addresses (all branches identical)

Defined in `config/conf_eeprom.h`:

| Axis | Kp addr | Ki addr | Kd addr |
|---|---|---|---|
| X | 0x4F (79) | 0x53 (83) | 0x57 (87) |
| Y | 0x81 (129) | 0x85 (133) | 0x89 (137) |
| Z | 0xB3 (179) | 0xB7 (183) | 0xBB (187) |

- Stride: 50 bytes (0x32) per axis
- Format: 4-byte IEEE 754 floats
- Tuning flag: byte 0x30 (48), bitmask of tuned axes

mc-refactor does NOT change these addresses. The refactor's `pid__Instance` structs are instantiated at init via `follower__init()` from the same EEPROM, so address compatibility is preserved.

---

## Verdict

| Branch | PID rating | Recommendation |
|---|---|---|
| **mc-refactor** | 5/5 architecturally — gold reference | Study it; finish it (~20 engineer-days); port cascade into a blessed branch |
| master / v4.2e / all tags | 2/5 — functional but monolithic | Use for production today; not a long-term architecture |
| hotFix | 2/5 (tuning UI removed) | Deploy-only variant once gains locked |
| referencePointsFix | 2/5 | Focus is backlash anticipation, not PID changes |
| lookahead | 2/5 | Trajectory planning, PID unchanged |

**Data source:** 3-agent swarm audit, 2026-04-20, against `whatbox-cnc-firmware-v4.1` at fetch-all state (all 29 branches + 12 tags locally).
