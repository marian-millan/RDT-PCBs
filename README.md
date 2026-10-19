# Lunabotics Rover Electronics

Custom PCBs for our NASA Lunabotics rover: a **Teensy 4.1 control board** that connects the autonomy computer to every actuator and sensor, and a **power distribution board** that supplies, protects, and monitors power across the rover.

Both boards were designed in KiCad and validated through full-rover integration testing with the mechanical and software subsystems (Spring 2026).

---

## System Overview

```
                   ┌────────────────────┐
                   │  Jetson Orin Nano  │
                   └─────────┬──────────┘
                             │ I²C
                   ┌─────────┴──────────┐
                   │   Control Board    │
                   │    (Teensy 4.1)    │
                   └─────────┬──────────┘
                             │
         ┌───────────────────┼────────────────────┐
         │ CAN               │ drivers            │ I/O
┌────────┴───────┐  ┌────────┴───────┐  ┌─────────┴────────┐
│      NEO       │  │   Stepper +    │  │  Load cells ×4   │
│   brushless    │  │   vibration    │  │   Encoders ×2    │
│     motors     │  │     motors     │  │  String pots ×2  │
└────────────────┘  └────────────────┘  │ Current sense ×8 │
                                        └──────────────────┘

  ┌────────────────┐    ┌────────────────┐
  │  24 V battery  │    │  12 V battery  │
  └────────┬───────┘    └────────┬───────┘
           └──────────┬──────────┘
          ┌───────────┴──────────┐
          │     Relay Board      │
          │  E-stop: cuts 24 V   │
          └───────────┬──────────┘
      ┌───────────────┴──────────────┐
      │      Power Distribution      │
      │         Board (PDB)          │
      │       current sense ×8       │
      └─────┬─────────────────────┬──┘
            │ 24 V                │ 12 V
  ┌─────────┴────────┐  ┌─────────┴────────┐
  │     Motors &     │  │     Control      │
  │    actuators     │  │   electronics    │
  └──────────────────┘  └──────────────────┘
```

---

## Control Board (Teensy 4.1)

Two-layer PCB built around a Teensy 4.1, acting as the bridge between the Jetson Orin Nano and the rover's hardware.

### Features
- **Jetson link over I²C** for commands and telemetry
- **Single shared CAN bus** for all NEO brushless motors (locomotion and excavation)
- **On-board stepper and vibration-motor drivers**, reducing internal wiring in the electronics box
- **Closed-loop feedback inputs** for autonomous excavation and deposition

### Sensors

| Sensor | Qty | Purpose |
|---|---|---|
| Load cells | 4 | Collection bin mass |
| Rotary encoders | 2 | Wheel odometry |
| String potentiometers | 2 | Excavator depth, deposition gate position |
| Current-sense channels (via mux) | up to 8 | Stall / overcurrent detection (from power board) |

---

## Power Distribution Board

Distributes power from the rover's two batteries and monitors per-channel current. Both batteries pass through a relay board before reaching the PDB.

### Power Path

| Source | Path | Loads |
|---|---|---|
| 24 V battery | Relay board → PDB | Motors and actuators |
| 12 V battery | Relay board → PDB | Control electronics (Teensy, Jetson, sensors) |

### Features
- **24 V distribution** to motors and actuators
- **12 V distribution** to control electronics
- **Reverse-polarity and overcurrent protection**
- **Per-channel current sensing**, 0–20 A on up to 8 channels
- **E-stop** on the relay board that removes 24 V motor power while keeping compute and sensors powered, so the rover stays observable after a stop

### Current Sensing

Sensing uses a **1 mΩ shunt resistor + analog current-sense amplifier** per channel. This approach was chosen over digital power-monitor ICs and Hall-effect sensors through a weighted trade study.

| Parameter | Value |
|---|---|
| Shunt resistance | 1 mΩ |
| Sense range | 0–20 A per channel |
| Shunt voltage at 20 A | 20 mV |
| Shunt dissipation at 20 A | ~0.4 W |
| Channels | up to 8 |

Amplified outputs are routed through an **analog multiplexer** into the Teensy ADC. Firmware uses these readings to detect motor stalls and overcurrent events and to transition the rover into a safe state.

### Energy Budget

| | Value |
|---|---|
| Budget per 15-min run | 31 Wh |
| Measured per 15-min run | 29.7 Wh |

---

## Repository Structure

```
.
├── control-board/
│   ├── kicad/          # schematic + PCB layout
│   ├── gerbers/        # fabrication outputs
│   └── bom/            # bill of materials
├── power-board/
│   ├── kicad/
│   ├── gerbers/
│   └── bom/
├── firmware/           # Teensy firmware
└── README.md
```

---

## Getting Started

### Opening the designs
1. Install [KiCad](https://www.kicad.org/) (version: _TODO_).
2. Open `control-board/kicad/*.kicad_pro` or `power-board/kicad/*.kicad_pro`.

### Fabrication
Gerbers and drill files are in each board's `gerbers/` folder. Both boards are two-layer.

### Firmware
_TODO: build/upload instructions (Arduino IDE), Jetson I²C address, CAN bus settings._

---

## Bring-Up & Testing

Both boards were assembled, brought up, and validated in full-rover integration testing with the mechanical and software teams, including:
- CAN communication with all NEO motors
- Sensor feedback during autonomous excavation and deposition
- Stall/overcurrent detection and safe-state transitions
- E-stop behavior (24 V cut, 12 V logic retained)
- Energy use over a full 15-minute run

---

## Team

Designed by the NYU Tandon RDT electrical team.
- Owen Chang
- Elizaveta Koroza
- Yuyan Luo
- Marian Millan
- Josue Munoz
- Jason Qin
- Eric Wang
- Bonnie Wu

## License

_TODO: add a license_
