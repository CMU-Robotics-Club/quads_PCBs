# quads_PCBs

PCB designs for Laika, the CMU Robotics Club quadruped. Software lives in
[Laika-Software](https://github.com/CMU-Robotics-Club/Laika-Software).

EDA tool: **KiCad**.

## Boards

| Board | Status | Purpose |
|---|---|---|
| `mainboard` | Planning | STM32H723 bridge between the Jetson and the 12 leg motors |

## System overview

```
Jetson Orin Nano
      │ USB
      ▼
┌──────────── mainboard (this repo) ────────────┐
│ STM32H723                                     │
│  ├─ FDCAN1 ─ transceiver ─────────────────────┼─► 4× hip ab/ad (AK80-8)
│  ├─ FDCAN2 ─ transceiver ─────────────────────┼─► 4× thigh
│  ├─ FDCAN3 ─ transceiver ─────────────────────┼─► 4× knee
│  ├─ SPI ─ MCP2518FD (not populated in v1) ────┼─► reserved
│  ├─ SPI ─ IMU                                 │
│  └─ GPIO ─ E-stop input                       │
│ Power: battery → 5 V / 3.3 V                  │
└───────────────────────────────────────────────┘
```

See [docs/architecture.md](docs/architecture.md) for the interface spec.
