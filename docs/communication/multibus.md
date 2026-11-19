---
layout: default
title: "Multibus Register Protocol"
nav_order: 3
parent: "Communication & GUI"
permalink: /communication/multibus/
---

# Multibus Register Protocol (`barefoc-multibus`)

`barefoc-multibus` provides a descriptor-driven control table framework for address-based buses such as I2C, SPI, and I3C. It is modeled after the memory-mapped register tables common in industrial servomotors and robot actuators.

---

## Memory-Mapped Control Table

The server core (`RegMap`) organizes motor parameters, setpoints, and telemetry into indexed registers:

| Address Offset | Name | Type | Access | Description |
|---|---|---|---|---|
| `0x00` | `DEVICE_ID` | `u16` | RO | Device model identifier |
| `0x02` | `FW_VERSION` | `u16` | RO | Firmware revision |
| `0x10` | `CONTROL_MODE` | `u8` | RW | `0`=Idle, `1`=Torque, `2`=Velocity, `3`=Angle |
| `0x14` | `TARGET_VALUE` | `f32` | RW | Commanded target setpoint |
| `0x20` | `MEASURED_ANGLE` | `f32` | RO | Actual shaft mechanical angle |
| `0x24` | `MEASURED_VELOCITY` | `f32` | RO | Actual shaft velocity |
| `0x30` | `BUS_VOLTAGE` | `f32` | RO | Inverter DC supply voltage |
| `0xF0` | `COMMAND_TRIGGER` | `u8` | WO | Self-clearing actions (`1`=Zero, `2`=Reset, `3`=Bootloader) |

---

## Key Features

- **Address Latching & Auto-Increment**: A single register write sets the internal address pointer; subsequent byte reads automatically increment the pointer, enabling high-bandwidth burst transfers of contiguous blocks (e.g., reading angle, velocity, and current in one I2C transaction).
- **Self-Clearing Command Registers**: Writing an action code triggers a hardware callback (such as re-zeroing the encoder or resetting into bootloader mode), after which the register immediately resets to zero.
- **Controller-Side Client Abstractions**: Includes `RegClientI2c` and `RegClientSpi` for master controllers to query motor nodes with type safety.
