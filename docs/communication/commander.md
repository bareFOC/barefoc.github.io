---
layout: default
title: "Commander Protocol"
nav_order: 1
parent: "Communication & GUI"
permalink: /communication/commander/
---

# Commander Protocol (`barefoc-commander`)

`barefoc-commander` is a pure-Rust, `#![no_std]` ASCII command processor implementing the SimpleFOC `commands.h` grammar and extending it with multi-axis robotics kinematics. It operates generically over any byte stream implementing `embedded_io::Read` and `embedded_io::Write` (UART, USB CDC, RTT, or TCP).

---

## 1. Single-Motor Command Grammar

A motor command consists of an identifier character, an optional sub-command, and a floating-point value:

```
[Command][SubCommand] [Value]\n
```

### Motor Command Reference:

| Command | Parameter / Action | Example | Response |
|:---:|---|---|---|
| `T` | Set target motion setpoint (V, rad/s, rad) | `T 20.5` | Target updated to 20.5 |
| `M` | Change motion control mode (`0`=torque, `1`=velocity, `2`=angle) | `M 1` | Velocity mode active |
| `C` | Change torque control mode (`0`=voltage, `1`=dc_curr, `2`=foc_curr) | `C 0` | Voltage mode active |
| `V` | Velocity PID tuning (`P`, `I`, `D`, `R`, `F`) | `VP 0.1` | Set velocity P gain to 0.1 |
| `A` | Angle PID tuning (`P`, `I`, `D`, `R`, `F`) | `AP 15.0` | Set angle P gain to 15.0 |
| `L` | Limit configuration (`V`=voltage, `C`=current, `S`=velocity) | `LV 5.0` | Voltage ceiling = 5.0 V |
| `E` | Motor state (`0`=disable, `1`=enable) | `E 1` | Motor enabled |

---

## 2. Robotics Kinematics Command Hierarchy (`KinematicsCommander`)

The dedicated `'K'` command identifier exposes complete interactive console tuning, telemetry streaming, dynamic sensor channel routing, and calibration sequencing for multi-axis robotic mechanisms.

Commands can be invoked directly (e.g. `KT 0 1.57`) or prefixed with `K` under a commander router (e.g. `K KT 0 1.57`):

```
K[SubCommand] [JointIndex] [Value]\n
```

### A. Topology & Sensor Routing Subcommands

| Command | Arguments | Purpose | Example | Description |
|---|---|---|---|---|
| `KM` | `<mode>` | Topology preset mode | `KM 1` | Sets topology preset: `0`=Four1Dof, `1`=Two2Dof, `2`=One1DofOne2Dof, `3`=Two1DofOne2Dof, `255`=Custom. Bare `KM` queries active mode. |
| `KS` | `<mot>:<ch>` / `<mot>` | Sensor routing map | `KS 0:2` | Assigns sensor channel 2 to motor axis 0. `KS 0` queries motor 0 routing; bare `KS` prints all 4 sensor mappings. |
| `KX` | `<mask>` | 2-DOF coupling bitmask | `KX 3` | Sets bitmask indicating coupled differential joint pairs (e.g. `0x03` couples axes [0, 1] and [2, 3]). |

---

### B. Joint Targets & Real-Time Telemetry

| Command | Arguments | Purpose | Example | Description |
|---|---|---|---|---|
| `KT` | `[<j>] [<val>]` | Joint target position | `KT 0 1.5708` | Sets target position setpoint for joint `j` (in radians). Bare `KT` prints targets for all joints. |
| `KA` | `[<j>]` | Joint feedback angle | `KA 0` | Reads measured joint angle from the assigned sensor channel (radians). |
| `KV` | `[<j>]` | Joint feedback velocity | `KV 0` | Reads measured joint velocity from the assigned sensor channel (rad/s). |

---

### C. Transmission Parameters

| Command | Arguments | Purpose | Example | Description |
|---|---|---|---|---|
| `KG` | `<j> <val>` | Gear reduction ratio | `KG 0 50.0` | Sets kinematic gear ratio ($R$) for joint `j`. |
| `KB` | `<j> <val>` | Backlash deadband amplitude | `KB 0 0.003` | Sets backlash half-deadband $\delta$ (radians) for joint `j`. |
| `KW` | `<j> <val>` | Velocity smoothing threshold | `KW 0 0.05` | Sets smooth tanh transition threshold $\omega_{\text{thresh}}$ (rad/s). |
| `KK` | `<j> <val>` | Torsional stiffness | `KK 0 1500.0` | Sets torsional compliance $K_{\text{gear}}$ (Nm/rad). |
| `KZ` | `<j> <val>` | Joint zero offset | `KZ 0 0.125` | Sets mechanical zero offset (radians) for joint `j`. |

---

### D. Outer Cascaded Joint PID Regulators

| Command | Arguments | Purpose | Example | Description |
|---|---|---|---|---|
| `KP` | `<j> <val>` | Proportional gain ($K_p$) | `KP 0 25.0` | Sets outer position loop proportional gain. |
| `KI` | `<j> <val>` | Integral gain ($K_i$) | `KI 0 0.5` | Sets outer position loop integral gain. |
| `KD` | `<j> <val>` | Derivative gain ($K_d$) | `KD 0 0.08` | Sets outer position loop derivative gain. |
| `KL` | `<j> <val>` | Velocity output limit | `KL 0 15.0` | Clamps commanded velocity output to $\pm 15.0$ rad/s. |
| `KR` | `<j> <val>` | Acceleration ramp limit | `KR 0 100.0` | Caps outer loop output acceleration ramp to $100.0\ \text{rad/s}^2$. |
| `KN` | `<j> <int>` | Loop decimation factor | `KN 0 10` | Decimates outer position loop to execute once every 10 inner FOC ticks. |

---

### E. Calibration Sequencing & Flash Status

| Command | Arguments | Purpose | Description |
|---|---|---|---|
| `KC` | `<cmd>` | Calibration action | Dispatches calibration commands: `KC 1` = Start stall sweep / autonomous limit search, `KC 2` = Abort procedure, `KC 3` = Save calibration to flash, `KC 4` = Load calibration from flash, `KC 8` = Reset state machine to idle. Bare `KC` returns current state (`0`=Idle, `1`=Running, `2`=Done, `3`=Error). |
| `KF` | `[<status>]` | Flash CRC32 status | Queries or sets flash calibration validity (`1`=Valid and CRC32 verified, `0`=Uninitialized or CRC mismatch). |

---

## 3. Four-Axis Assembly Dispatcher (`Commander4Axis`)

`barefoc-commander` provides the `Commander4Axis` helper, which binds motor axes `'A'`, `'B'`, `'C'`, `'D'` and kinematics `'K'` under a single command router:

```
┌────────────────────────────────────────────────────────┐
│ Incoming ASCII Command Stream (e.g. "B T 10.0\n")      │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
           ┌──────────────────────────────────┐
           │ Commander4Axis Command Router    │
           └──────┬───────┬───────┬───────┬───┘
                  │       │       │       │   'K'
      'A'         │   'B' │   'C' │   'D' │    │
       ▼          ▼       ▼       ▼       ▼    ▼
    [Motor A]  [Motor B] [Motor C] [Motor D] [Kinematics]
```

### Rust Example:

```rust,no_run
use barefoc_commander::Commander4Axis;
use barefoc::motor::base::NullClock;

let clock = NullClock;
let mut cmd4 = Commander4Axis::new_with_clock(&clock);

// 1. Command motor B to 10 rad/s velocity
cmd4.run("B T 10.0\n");

// 2. Set kinematics topology to Two2Dof (2x 2-DOF differential gimbals)
cmd4.run("K KM 1\n");

// 3. Route sensor pool channel 3 to motor axis 2
cmd4.run("K KS 2:3\n");

// 4. Command target position for joint 0 to 1.57 rad
cmd4.run("K KT 0 1.57\n");

// 5. Query measured joint angle on joint 0
cmd4.run("K KA 0\n");
```

---

## 4. Output Verbosity Modes

Output formatting across both motor and kinematics commands can be adjusted via `VerboseMode`:

- **`Nothing`**: Completely silent execution (zero byte overhead).
- **`Minimal`**: Returns only raw numeric values (ideal for embedded microcontrollers and lightweight parsers).
- **`UserFriendly`**: Returns descriptive labels and engineering units (e.g., `Kinematics| Mode: Two2Dof, J0 target: 1.570 rad`).
- **`MachineReadable`**: Formatted key-value pairs (`K:KM=1`, `K:KT0=1.570`) for host plotting tools and telemetry loggers.

---

## 5. Smart Servo Console Interface (`SmartServoCommander`)

For actuators configured with the unified `ControlTable`, `barefoc-commander` provides `SmartServoCommander`. It parses high-level servo operations, multi-motor setpoints, kinematics presets, tuning gains, and bootloader triggers over ASCII streams:

### Command Grammar Reference:

| Command Pattern | Action | Example | Description |
|---|---|---|---|
| `M<id> C <mode>` | Operating mode switch | `M0 C 2` | Switches motor 0 to Position mode (0–7) |
| `M<id> T <val>` | Target setpoint | `M0 T 3.14` | Sets goal position / velocity setpoint |
| `K M <preset>` | Topology preset | `K M 0` | Sets kinematics preset (`0`=4x1DOF, `1`=2x2DOF) |
| `K T <joint> <val>` | Kinematic joint target | `K T 0 1.57` | Commands outer joint position |
| `P <gain>` | Position loop gain | `P 25.0` | Writes to `PositionPGain` register (`0x34`) |
| `V <gain>` | Velocity loop gain | `V 0.5` | Writes to `VelocityPGain` register (`0x40`) |
| `SAVE` | Flash commit | `SAVE` | Triggers `SaveConfigCommand` to commit EEPROM to flash |
| `RELOAD` | Flash reload | `RELOAD` | Reloads non-volatile settings from flash |
| `FACTORY` | Factory reset | `FACTORY` | Restores default parameters and overwrites flash |
| `DIAG` | Status query | `DIAG` | Returns present voltage, current, temperature, and errors |
| `!BOOT` | Bootloader jump | `!BOOT` | Writes mailbox cookie and reboots into Katapult |

---

## 6. Lightweight Table-Free Float Parser

Standard Rust `core::str::FromStr` for floating-point values pulls in the `core::num::dec2flt` module, which contains a **10.4 KiB** lookup table (`POWER_OF_FIVE_128`). In resource-constrained microcontrollers with strict 104 KiB application flash limits, this table consumes ~10% of total flash storage.

`barefoc-commander` includes a table-free, `#![no_std]` ASCII float parser (`barefoc_commander::parse_f32`):

```rust,no_run
use barefoc_commander::parse_f32;

// Parses sign, integer digits, and fractional digits with 64-bit mantissa scaling
let value: f32 = parse_f32("-12.345").expect("Valid float");
assert_eq!(value, -12.345);
```

### Flash Footprint Comparison:

| Parser Implementation | Flash Overhead | Dec2Flt Table | Standard Compliance |
|---|:---:|:---:|---|
| Standard `core::str::FromStr` | $\approx 14.8\ \text{KiB}$ | Included ($10.4\ \text{KiB}$) | IEEE 754 full accuracy |
| `barefoc_commander::parse_f32` | $\approx 380\ \text{bytes}$ | **Zero ($0\ \text{bytes}$)** | 64-bit mantissa scaled to `f32` |

By replacing `FromStr` with `parse_f32`, applications reclaim over **14 KiB** of flash storage with identical setpoint accuracy.
