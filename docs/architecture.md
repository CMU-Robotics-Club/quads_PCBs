# Mainboard architecture

Status: **draft**.

## Scope

The mainboard replaces the Nucleo-H723ZG and breakout modules with one PCB:

- STM32H723 MCU
- 3 CAN buses to the motors (on-chip FDCAN1–3), with CAN-FD capable
  transceivers (≥ 5 Mbps)
- USB to the Jetson Orin Nano
- IMU
- E-stop / power switch input
- Power: its own supply from the battery (5 V / 3.3 V), not from the Jetson

Not on this board: the motors, the Jetson, and motor power. Motors get power
straight from the battery.

## Motors

- 12× CubeMars AK80-8 KV30 motors, 3 per leg
- Each motor has an ODrive S1 driver
- Protocol: ODrive CAN, classic CAN frames
  ([code](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_hardware_interface/odrive_base/src/socket_can.cpp#L106)),
  1 Mbps
  ([code](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_hardware_interface/hardware/laika_hardware_interface.cpp#L80))
- ODrive S1 supports CAN-FD from firmware 0.6.10
  ([ODrive docs](https://docs.odriverobotics.com/v/latest/hardware/odrive-comparison.html)).
  Laika-Software currently uses classic CAN.

## Control rate

**1 kHz.** [Laika-Software](https://github.com/CMU-Robotics-Club/Laika-Software)
runs its controllers at
[`update_rate: 1000`](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_pid_controller/config/real_leg_pid_controller_config.yaml#L3).

Every cycle, the host sends 3 frames to each motor:

1. [`Set_Input_Torque`](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_hardware_interface/hardware/laika_hardware_interface.cpp#L287): the torque command
2. [`Get_Torques`](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_hardware_interface/hardware/laika_hardware_interface.cpp#L251): a request for torque data
3. [`Get_Encoder_Estimates`](https://github.com/CMU-Robotics-Club/Laika-Software/blob/dd9dcf645790c05628127b254bd854ea58fc05dc/laika_ws/src/laika_hardware_interface/hardware/laika_hardware_interface.cpp#L252): a request for position and velocity

## CAN bus plan

Grouped by joint type, 4 motors per bus:

| Bus | Controller | Motors |
|---|---|---|
| CAN1 | FDCAN1 | 4× hip ab/ad (one per leg) |
| CAN2 | FDCAN2 | 4× thigh |
| CAN3 | FDCAN3 | 4× knee |

Wiring: each bus runs from motor to motor in a chain, with a 120 Ω resistor
at both ends (one on this board, one at the last motor).

## Jetson link

USB Full-Speed.
