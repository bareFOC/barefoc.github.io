---
layout: default
title: "Commander Protocol"
nav_order: 1
parent: "Communication & GUI"
permalink: /communication/commander/
---

# Commander Protocol (`barefoc-commander`)

`barefoc-commander` is a pure-Rust, `#![no_std]` ASCII command processor implementing the SimpleFOC `commands.h` grammar. It operates generically over any byte stream implementing `embedded_io::Read` and `embedded_io::Write` (UART, USB CDC, RTT, or TCP).

---

## Command Grammar

A command consists of an identifier character, an optional sub-command, and a floating-point value:

```
[Command][SubCommand] [Value]\n
```

### Motor Command Reference:

| Command | Parameter / Action | Example | Response |
|---|---|---|---|
| `T` | Set target motion setpoint (V, rad/s, rad) | `T 20.5` | Target updated to 20.5 |
| `M` | Change motion control mode (`0`=torque, `1`=velocity, `2`=angle) | `M 1` | Velocity mode active |
| `C` | Change torque control mode (`0`=voltage, `1`=dc_curr, `2`=foc_curr) | `C 0` | Voltage mode active |
| `V` | Velocity PID tuning (`P`, `I`, `D`, `R`, `F`) | `VP 0.1` | Set velocity P gain to 0.1 |
| `A` | Angle PID tuning (`P`, `I`, `D`, `R`, `F`) | `AP 15.0` | Set angle P gain to 15.0 |
| `L` | Limit configuration (`V`=voltage, `C`=current, `S`=velocity) | `LV 5.0` | Voltage ceiling = 5.0 V |
| `E` | Motor state (`0`=disable, `1`=enable) | `E 1` | Motor enabled |

---

## Output Verbosity Modes

Output formatting can be adjusted via `VerbosityMode`:

- **`Nothing`**: Completely silent (zero byte overhead).
- **`Minimal`**: Returns only raw numeric feedback.
- **`UserFriendly`**: Returns descriptive labels (e.g., `PID vel| P: 0.100, I: 1.000`).
- **`MachineReadable`**: Formatted key-value pairs for automated parsing scripts.

---

## Rust Integration Example

```rust,no_run
use barefoc_commander::{Commander, LineProcessor};
use barefoc_commander::motor::MotorCommander;
use embedded_io::{Read, Write};

pub fn poll_serial<IO: Read + Write>(io: &mut IO, motor: &mut BLDCMotor) {
    let mut line_buf = [0u8; 64];
    let mut processor = LineProcessor::new(&mut line_buf);

    let mut commander = Commander::new();
    commander.add_motor('M', motor);

    // Reads incoming bytes from UART / RTT, parses lines, and dispatches commands
    processor.process(io, &mut commander);
}
```
