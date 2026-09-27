# Mainboard architecture

Status: **draft** — decisions below are proposals until the open questions
are closed.

## Scope

The mainboard integrates what is currently a Nucleo-H723ZG plus breakout
modules into one PCB:

- STM32H723 MCU
- 3 CAN buses to the motors (grouped by joint type), plus a footprint for a
  4th (MCP2518FD, not populated in v1)
- USB to the Jetson Orin Nano
- IMU
- E-stop / power switch input
- Power: battery → 5 V / 3.3 V

Out of scope: the motors, the Jetson, and motor power distribution (motors
are powered from the battery directly, not through this board's regulators).

## Motors

12× CubeMars (T-Motor) AK80-8 KV30, 3 per leg, driven in MIT mode.

- Bus: **classic CAN, 1 Mbps**. Not CAN-FD — no FD frames may be sent on a
  bus with these motors. *(Verify against the AK80-8 manual.)*
- Each control cycle: 1 command frame + 1 reply frame per motor.

## CAN bus plan

Grouped by joint type, 4 motors per bus:

| Bus | Controller | Motors |
|---|---|---|
| CAN1 | FDCAN1 (on-chip, classic mode) | 4× hip ab/ad (one per leg) |
| CAN2 | FDCAN2 (on-chip, classic mode) | 4× hip pitch / thigh |
| CAN3 | FDCAN3 (on-chip, classic mode) | 4× knee |
| CAN4 | MCP2518FD over SPI — **footprint only, not populated in v1** | reserved |

Bus load: a classic 8-byte frame at 1 Mbps takes roughly 130 µs (estimate
incl. bit stuffing). Per bus, 4 motors × 2 frames = 8 frames per cycle.

| Rate | Bus load (4 motors/bus) |
|---|---|
| 500 Hz | ≈ 52 % |
| 1 kHz | ≈ 104 % (impossible) |

Target control rate: **500 Hz**. Reaching 1 kHz requires populating CAN4
and switching to one bus per leg (3 motors/bus, ≈ 78 % at 1 kHz).

Wiring: each bus is a daisy chain through its 4 motors (no star/stub
topology), with a 120 Ω termination at both ends (board end on this PCB,
far end at the last motor).

## Jetson link

USB Full-Speed (CDC). Latency jitter must be measured before committing to
rates above 500 Hz. Fallback: SPI or UART from the Jetson 40-pin header.

## Open questions

- [ ] Battery voltage (sets regulator choice and connector ratings)
- [ ] H723 package: LQFP144 (ZG, same as Nucleo) or smaller (e.g. LQFP100)
- [ ] CAN transceiver part (3.3 V logic, ≥ 1 Mbps)
- [ ] IMU part and SPI port
- [ ] E-stop behavior: MCU input only, or also hardware cut of motor power
- [ ] Confirm AK80-8 has no internal 120 Ω termination
- [ ] Connector types for CAN, USB, power, E-stop
- [ ] Board size / mounting holes (from the mechanical team)
- [ ] Motor mounting locations (affects CAN cable routing for joint-type grouping)
