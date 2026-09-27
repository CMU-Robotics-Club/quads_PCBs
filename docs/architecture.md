# Mainboard architecture

Status: **draft**.

## Scope

The mainboard integrates what is currently a Nucleo-H723ZG plus breakout
modules into one PCB:

- STM32H723 MCU
- 3 CAN buses to the motors (on-chip FDCAN1–3, grouped by joint type)
- USB to the Jetson Orin Nano
- IMU
- E-stop / power switch input
- Power: battery → 5 V / 3.3 V

Out of scope: the motors, the Jetson, and motor power distribution (motors
are powered from the battery directly, not through this board's regulators).

## Motors

12× CubeMars (T-Motor) AK80-8 KV30, 3 per leg.

**Driver/protocol not confirmed.** The current Laika-Software hardware
interface uses the ODrive CAN protocol (torque control).

- Bus: **classic CAN, 1 Mbps**. Not CAN-FD — no FD frames may be sent on a
  bus with these motors. *(Verify against the AK80-8 manual.)*
- Each control cycle: at least 1 command frame + 1 reply frame per motor
  (exact count depends on the protocol).

## CAN bus plan

Grouped by joint type, 4 motors per bus:

| Bus | Controller | Motors |
|---|---|---|
| CAN1 | FDCAN1 (on-chip, classic mode) | 4× hip ab/ad (one per leg) |
| CAN2 | FDCAN2 (on-chip, classic mode) | 4× hip pitch / thigh |
| CAN3 | FDCAN3 (on-chip, classic mode) | 4× knee |

Bus load: a classic 8-byte frame at 1 Mbps takes roughly 130 µs (estimate
incl. bit stuffing). Per bus, assuming 2 frames per motor: 4 motors × 2 = 8
frames per cycle.

| Rate | Bus load (4 motors/bus) |
|---|---|
| 500 Hz | ≈ 52 % |
| 1 kHz | ≈ 104 % (impossible) |

Target control rate: **1 kHz**. Laika-Software runs its controllers at
`update_rate: 1000` with the PID loop on the host, sending torque commands
every cycle.

**The 3-bus plan above does not meet 1 kHz.** Bus plan is pending the final
motor/driver choice.

Wiring: each bus is a daisy chain through its 4 motors (no star/stub
topology), with a 120 Ω termination at both ends (board end on this PCB,
far end at the last motor).

## Jetson link

USB Full-Speed (CDC). Latency jitter must be measured to confirm it supports
1 kHz. Fallback: SPI or UART from the Jetson 40-pin header.
