---
layout: default
title: "Fibre RPC & ODrive GUI"
nav_order: 2
parent: "Communication & GUI"
permalink: /communication/fibre_gui/
---

# Fibre RPC & ODrive GUI (`barefoc-fibre`)

`barefoc-fibre` implements a zero-allocation, `#![no_std]` server for the **Fibre RPC protocol**, allowing any BareFOC device to connect natively to the official [ODrive Web GUI](https://gui.odriverobotics.com) and the `odrivetool` Python CLI.

---

## 1. Why Fibre?

Traditional serial consoles require custom desktop GUI applications for plotting curves. The ODrive Web GUI runs directly in modern Chromium-based browsers via WebUSB or WebSerial, providing:
- Real-time oscilloscope plotting of position, velocity, and phase currents.
- Interactive parameter sliders for PID tuning with immediate visual feedback.
- Automated system identification and step-response profiling.
- Multi-axis robotic actuator monitoring and live kinematics calibration.

```
┌─────────────────────────────────┐
│ Browser: ODrive Web GUI         │
│ (https://gui.odriverobotics.com)│
└────────────────┬────────────────┘
                 │ WebSerial / WebUSB / TCP
                 ▼
┌─────────────────────────────────┐
│ Fibre Protocol Engine           │
│ - Packet Framing & CRC-16       │
│ - Endpoint 0 JSON Schema        │
│ - Variable Read / Write / Stream│
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│ BareFOC ActuatorAssembly State  │
│ 4 Motors • 4 Joints • Calibrator│
└─────────────────────────────────┘
```

---

## 2. 4-Axis Kinematics Schema (`BAREFOC_4AXIS_KINEMATICS_JSON`)

Upon initial handshake, the host GUI requests **Endpoint 0**. `barefoc-fibre` returns the compressed JSON schema `BAREFOC_4AXIS_KINEMATICS_JSON`, defining 58 structured endpoints covering system supply voltage, topology presets, motor commutation telemetry, outer joint position controllers, and on-device auto-calibration.

### Schema Hierarchy & Endpoint Map (58 Endpoints):

```json
[
  {"name": "vbus_voltage", "id": 1, "type": "float", "access": "r"},
  {"name": "topology_mode", "id": 2, "type": "uint32", "access": "rw"},
  {
    "name": "axis0",
    "type": "object",
    "members": [
      {"name": "target", "id": 3, "type": "float", "access": "rw"},
      {"name": "current", "id": 4, "type": "float", "access": "r"},
      {"name": "shaft_angle", "id": 5, "type": "float", "access": "r"},
      {"name": "shaft_velocity", "id": 6, "type": "float", "access": "r"},
      {"name": "sensor_channel", "id": 7, "type": "uint32", "access": "rw"}
    ]
  },
  {
    "name": "joint0",
    "type": "object",
    "members": [
      {"name": "target_pos", "id": 23, "type": "float", "access": "rw"},
      {"name": "present_pos", "id": 24, "type": "float", "access": "r"},
      {"name": "present_vel", "id": 25, "type": "float", "access": "r"},
      {"name": "gear_ratio", "id": 26, "type": "float", "access": "rw"},
      {"name": "backlash", "id": 27, "type": "float", "access": "rw"},
      {"name": "pid_p", "id": 28, "type": "float", "access": "rw"},
      {"name": "pid_i", "id": 29, "type": "float", "access": "rw"},
      {"name": "pid_d", "id": 30, "type": "float", "access": "rw"}
    ]
  },
  {
    "name": "calib",
    "type": "object",
    "members": [
      {"name": "command", "id": 55, "type": "uint32", "access": "rw"},
      {"name": "state", "id": 56, "type": "uint32", "access": "r"},
      {"name": "progress", "id": 57, "type": "float", "access": "r"},
      {"name": "flash_status", "id": 58, "type": "uint32", "access": "r"}
    ]
  }
]
```

### Complete Endpoint Registry:

| Endpoint Range | Category | Parameters & Access | Description |
|:---:|---|---|---|
| `1` – `2` | **System Global** | `vbus_voltage` (RO float)<br/>`topology_mode` (RW uint32) | DC bus voltage and active kinematic topology preset (`0`=Four1Dof, `1`=Two2Dof, `2`=One1DofOne2Dof, `3`=Two1DofOne2Dof) |
| `3` – `7` | **Axis 0 (Motor)** | `target` (RW), `current` (RO), `shaft_angle` (RO), `shaft_velocity` (RO), `sensor_channel` (RW) | Inner motor commutation telemetry, velocity/torque command, and assigned sensor channel |
| `8` – `12` | **Axis 1 (Motor)** | `target` (RW), `current` (RO), `shaft_angle` (RO), `shaft_velocity` (RO), `sensor_channel` (RW) | Inner motor axis 1 telemetry and sensor mapping |
| `13` – `17` | **Axis 2 (Motor)** | `target` (RW), `current` (RO), `shaft_angle` (RO), `shaft_velocity` (RO), `sensor_channel` (RW) | Inner motor axis 2 telemetry and sensor mapping |
| `18` – `22` | **Axis 3 (Motor)** | `target` (RW), `current` (RO), `shaft_angle` (RO), `shaft_velocity` (RO), `sensor_channel` (RW) | Inner motor axis 3 telemetry and sensor mapping |
| `23` – `30` | **Joint 0 (Outer)** | `target_pos` (RW), `present_pos` (RO), `present_vel` (RO), `gear_ratio` (RW), `backlash` (RW), `pid_p` (RW), `pid_i` (RW), `pid_d` (RW) | Cascaded outer position loop setpoints, feedback tracking, transmission ratio, backlash, and PID gains |
| `31` – `38` | **Joint 1 (Outer)** | `target_pos`, `present_pos`, `present_vel`, `gear_ratio`, `backlash`, `pid_p`, `pid_i`, `pid_d` | Joint 1 cascaded outer position loop parameters |
| `39` – `46` | **Joint 2 (Outer)** | `target_pos`, `present_pos`, `present_vel`, `gear_ratio`, `backlash`, `pid_p`, `pid_i`, `pid_d` | Joint 2 cascaded outer position loop parameters |
| `47` – `54` | **Joint 3 (Outer)** | `target_pos`, `present_pos`, `present_vel`, `gear_ratio`, `backlash`, `pid_p`, `pid_i`, `pid_d` | Joint 3 cascaded outer position loop parameters |
| `55` – `58` | **Calibration** | `command` (RW uint32), `state` (RO uint32), `progress` (RO float), `flash_status` (RO uint32) | On-device auto-calibration control, state machine phase, percentage progress, and flash validity |

---

## 3. Endpoint Registration (`make_4axis_assembly_endpoints`)

The `make_4axis_assembly_endpoints()` constructor binds the 58 endpoints directly to the fields of `ActuatorAssembly` without heap allocations:

```rust
pub fn make_4axis_assembly_endpoints() -> [Endpoint<ActuatorAssembly>; 58]
```

Each endpoint is registered with zero-allocation accessors:

```rust
// Global System
Endpoint::readonly_f32(1, |a| a.vbus_voltage),
Endpoint::readwrite_u32(2, |a| a.topology_mode, |a, v| a.topology_mode = v),

// Axis 0
Endpoint::readwrite_f32(3, |a| a.axes[0].target, |a, v| a.axes[0].target = v),
Endpoint::readonly_f32(4, |a| a.axes[0].current),
Endpoint::readonly_f32(5, |a| a.axes[0].shaft_angle),
Endpoint::readonly_f32(6, |a| a.axes[0].shaft_velocity),
Endpoint::readwrite_u32(7, |a| a.axes[0].sensor_channel, |a, v| a.axes[0].sensor_channel = v),

// Joint 0
Endpoint::readwrite_f32(23, |a| a.joints[0].target_pos, |a, v| a.joints[0].target_pos = v),
Endpoint::readonly_f32(24, |a| a.joints[0].present_pos),
Endpoint::readonly_f32(25, |a| a.joints[0].present_vel),
Endpoint::readwrite_f32(26, |a| a.joints[0].gear_ratio, |a, v| a.joints[0].gear_ratio = v),
Endpoint::readwrite_f32(27, |a| a.joints[0].backlash, |a, v| a.joints[0].backlash = v),
Endpoint::readwrite_f32(28, |a| a.joints[0].pid_p, |a, v| a.joints[0].pid_p = v),
Endpoint::readwrite_f32(29, |a| a.joints[0].pid_i, |a, v| a.joints[0].pid_i = v),
Endpoint::readwrite_f32(30, |a| a.joints[0].pid_d, |a, v| a.joints[0].pid_d = v),

// Calibration
Endpoint::readwrite_u32(55, |a| a.calib.command, |a, v| a.calib.command = v),
Endpoint::readonly_u32(56, |a| a.calib.state),
Endpoint::readonly_f32(57, |a| a.calib.progress),
Endpoint::readonly_u32(58, |a| a.calib.flash_status),
```

---

## 4. Rust Integration Example

```rust,no_run
use barefoc_fibre::{
    FibreServer, make_4axis_assembly_endpoints,
    Endpoint0Handler, BAREFOC_4AXIS_KINEMATICS_JSON, ActuatorAssembly
};
use embedded_io::{Read, Write};

pub fn run_fibre_assembly<IO: Read + Write>(io: &mut IO, assembly: &mut ActuatorAssembly) {
    let endpoints = make_4axis_assembly_endpoints();
    let schema_handler = Endpoint0Handler::new(BAREFOC_4AXIS_KINEMATICS_JSON);

    let mut server = FibreServer::new(&endpoints, schema_handler);
    let mut rx_buf = [0u8; 256];
    let mut tx_buf = [0u8; 256];

    loop {
        // Poll and dispatch Fibre RPC packets from WebUSB / WebSerial
        server.poll(io, assembly, &mut rx_buf, &mut tx_buf);
    }
}
```

---

## 5. Smart Servo Fibre Schema & RPCs (`barefoc-fibre::servo`)

For actuators using the unified 176-byte `ControlTable`, `barefoc-fibre::servo` provides `create_smart_servo_schema()` and `create_smart_servo_endpoints()`.

This schema exposes:
- **`eeprom` Object**: `node_id`, `operating_mode`, `current_limit`, `velocity_limit`, `position_p`, `velocity_p`, `current_p`.
- **`ram` Object**: `torque_enable`, `goal_position`, `goal_velocity`, `goal_current`, `present_position`, `present_velocity`, `present_current`, `present_voltage`, `present_temp`, `hardware_error_status`.
- **RPC Functions**:
  - `save_configuration()`: Commits active EEPROM settings to on-chip Flash storage.
  - `reboot_bootloader()`: Hands off execution to the Katapult bootloader for over-the-air firmware update.

```rust,no_run
use barefoc_fibre::servo::{create_smart_servo_schema, create_smart_servo_endpoints, SmartServoFibreContext};
use barefoc_fibre::{Endpoint0Handler, FibreServer};
use barefoc::control_table::ControlTable;

let mut table = ControlTable::new();
let endpoints = create_smart_servo_endpoints();
let schema = create_smart_servo_schema();
let schema_handler = Endpoint0Handler::new(schema.as_str());

let mut server = FibreServer::new(&endpoints, schema_handler);
```
