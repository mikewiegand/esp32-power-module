# Power Module vFinal

**Status:** FROZEN  
**Scope:** Reusable solar + battery power module for ESP32-class devices  
**Change policy:** No architectural changes without explicit version bump

---

# ESP32-S3 Solar Power & Battery Architecture — V2

## Purpose
This document defines a single, reusable, solar-powered battery architecture for ESP32-S3 outdoor devices.
It consolidates prior drafts into one normalized reference with:
- one power chain
- one battery strategy
- one voltage-sensing method
- one wiring diagram
- one BOM

---

## System Overview (Standard)

**Target MCU:** Seeed XIAO ESP32-S3  
**Primary Use Case:** Outdoor, solar-powered, event-driven (camera / sensor) nodes  
**Sleep Mode:** Light sleep (fast wake)  
**Power Philosophy:** Oversize solar, moderate battery, voltage-aware behavior

---

## Power Chain (Canonical)

```
[ 6V 10W Solar Panel ]
            │
            ▼
     [ BQ24074 Charger ]
      (power-path)
            │
     ┌──────┴─────────┐
     │                │
 [ LiPo 3.7V ]     [ SYS / OUT ]
  (3700mAh)             │
                         ▼
                  [ 3.3V Buck ]
                         │
                    ESP32-S3
```

---

## Battery Strategy (Standard)

- Chemistry: 1S LiPo
- Nominal Voltage: 3.7 V
- Capacity: ~3700 mAh
- Cold Behavior: Expect 60–75% effective capacity at 25–30°F (temporary)

---

## Battery Voltage Measurement (Standard)

### Method
Resistor divider into ESP32 ADC (no fuel gauge IC).

### Parts
- R1: 200 kΩ (BAT → ADC)
- R2: 100 kΩ (ADC → GND)
- C1: 0.1 µF ceramic (ADC → GND)

### Wiring
```
BQ24074 BAT ── R1 (200k) ──┐
                           ├── GPIO1 (ADC)
GND ───────── R2 (100k) ──┘
                           │
                        C1 0.1µF
                           │
                          GND
```

Divider ratio: 3:1  
Divider current ≈ 14 µA

---

## Voltage-Based Behavior Policy

| VBAT | Behavior |
|------|----------|
| ≥ 3.9 V | Full operation |
| 3.7–3.9 V | Normal |
| 3.5–3.7 V | Capture only |
| < 3.5 V | Sleep only |

---

## Solar Sizing

- Panel: 6 V · 10 W · ETFE
- Expected winter recovery: ~600–1200 mAh/day

---

## Regulation

- 3.3 V buck regulator (≥1 A peak)
- TPS22919 load switch for peripherals (recommended)

---

## Complete BOM

### Core
- Seeed XIAO ESP32-S3
- BQ24074 solar charger
- Voltaic Systems 10W 6V ETFE panel
- 1S LiPo 3.7V ~3700mAh

### Battery Sense
- 200 kΩ resistor (through-hole)
- 100 kΩ resistor (through-hole)
- 0.1 µF ceramic capacitor (through-hole)

---

## Summary

6V/10W solar → BQ24074 → 3700mAh LiPo → 3.3V buck → XIAO ESP32-S3, with battery awareness via a 200k/100k divider.

---

## Simple Wiring Diagram (Fritzing-Style)

This diagram shows **functional wiring** (what connects to what),
not physical board photos. It is the authoritative wiring reference.

![Simple wiring diagram](sandbox:/mnt/data/fritzing_style_diagram_v1.png)

### Wiring Summary
- **Solar Panel → BQ24074 Solar IN**
- **BQ24074 BAT → LiPo Battery**
- **BQ24074 OUT → TSR-1-2433 VIN**
- **TSR-1-2433 3.3V → XIAO ESP32-S3 3V3**
- **All GND pins tied together**
- **Battery Sense:**  
  - BAT → 200k → GPIO1  
  - GPIO1 → 100k → GND  
  - 0.1µF from GPIO1 → GND

This section supersedes any board photos or pin screenshots.

---

## How to Read This System (Expanded Walkthrough)

**Left → Right = power flow**  
Think: *energy source → conditioning → regulation → MCU*.

### 1) Solar Panel (6 V)
- Connects **only** to BQ24074 Solar IN
- Supplies raw energy
- Never connects to the ESP32 or buck directly
- MCU has no electrical awareness of the panel

### 2) BQ24074 (Charger + Power-Path)
- **BAT → LiPo battery**
  - Handles safe charging and current limiting
- **OUT (SYS) → system rail**
  - Feeds the system from solar, USB, or battery automatically
- Performs seamless source switching with no firmware involvement

### 3) TSR-1-2433 (3.3 V Buck)
- **VIN ← BQ24074 OUT** (this is the only rail feeding the buck)
- **3.3 V OUT → XIAO ESP32-S3 3V3 pin**
- **GND shared** with charger and ESP32
- ESP32 never sees raw battery or solar voltage

### 4) Battery Sense (Measurement Only)
- **BAT (LIPO pin)** → 200 kΩ → ADC node → **GPIO1**
- ADC node → 100 kΩ → GND
- **0.1 µF ceramic** from ADC node → GND

Notes:
- This path is **read-only** (measurement only)
- It does not power anything
- Power flows from OUT; information flows from BAT

### 5) Ground
- One common GND for:
  - Solar charger
  - Buck
  - ESP32
  - Battery
  - Sense divider
- Prefer a solid ground plane or very short return paths

---

## Battery Sense Accuracy (Expectation Setting)

- Expected accuracy for voltage-band decisions: **~90%**
- Typical error after one-time calibration: **±0.05–0.15 V**
- This is sufficient for threshold-based behavior:
  - 3.9 V / 3.7 V / 3.5 V

This method is not intended to be a lab-grade voltmeter.
It is intentionally simple, robust, and reliable for solar + cold outdoor systems.

---

# Power Architecture – Canonical (v2.3)

This section supersedes all previous regulator-selection discussion.
It reflects the **final, reusable, brownout-resistant power design**.

## Canonical Power Stack

```
Solar Panel
    ↓
BQ24074 (charger + power-path)
    ↓
S13V30F5 (5V, 3A buck-boost)
    ↓
5V SYSTEM RAIL
    ├── Seeed XIAO ESP32-S3 (via 5V pin or USB-C)
    ├── Optional peripherals that accept 5V
    └── TSR-1-2433 → Quiet 3.3V Rail (optional)
```

### Why this stack is final
- Buck-boost stage absorbs **battery sag, cold, and burst load**
- 3A class handles **ESP + sensors + LED + GPS simultaneously**
- 5V rail matches the **XIAO ESP32-S3 native input design**
- Scales cleanly across future builds

---

## Powering the Seeed XIAO ESP32-S3

### Supported inputs
- ✅ USB-C
- ✅ 5V pin / VBUS

The onboard regulator generates 3.3V for the ESP32.

### Prohibited
- ❌ Feeding 5V into the 3V3 pin
- ❌ Back-feeding 3V3 while also powering via 5V/USB

**Rule:** choose exactly one power entry path for the XIAO.

---

## Using the TSR-1-2433 (Optional but Useful)

The TSR-1 remains valid as a **secondary, quiet 3.3V regulator**.

### Recommended use
Feed TSR-1 from the **5V system rail** and use it for:
- PIR sensor
- Low-noise sensors
- GPS modules
- Always-on logic

### Not recommended
- Using TSR-1 as the main ESP32 rail when multiple devices may run concurrently
- Feeding TSR-1 directly from the battery / SYS rail

---

## Current Budgeting (Reality-Based)

Typical peak contributors:
- ESP32-S3 Wi-Fi TX bursts: 600–900 mA
- mmWave presence sensor: 60–120 mA
- LED (small): 20–150 mA
- GPS: 20–60 mA (higher during acquisition)

Design is sized for **peak concurrency**, not averages.

---

## Required Decoupling

Minimum recommended:
- On 5V rail: **470–1000 µF bulk**
- Near ESP32: **470 µF + 0.1 µF**
- Near peripheral rail: **220 µF + 0.1 µF**

These caps are mandatory for transient stability.

---

## Reusability Statement

This architecture is intentionally:
- Over-provisioned
- Boring
- Repeatable

It is suitable as a **drop-in power module** for:
- Outdoor solar nodes
- Battery-backed ESP32 systems
- Multi-sensor, burst-load designs

No further regulator redesign should be required for this class of device.
