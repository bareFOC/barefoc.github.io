---
layout: default
title: "Cargo Configuration & Features"
nav_order: 1
parent: "Getting Started"
permalink: /getting_started/cargo_setup/
---

# Cargo Configuration & Features

BareFOC uses Cargo feature flags to remain lightweight, enabling only the hardware peripherals, motor topologies, and math implementations required for your specific application.

---

## Adding Dependencies

In your project's `Cargo.toml`, add `barefoc` alongside the peripheral crates you require:

```toml
[dependencies]
# Core FOC algorithms, transforms, and motor models
barefoc = { version = "0.1", default-features = false }

# Optional: Specialized sensors (AS5600, MT6701) and smart gate drivers (DRV8316, DRV8311)
barefoc-drivers = { version = "0.1", default-features = false }

# Optional: Telemetry & serial commanding (Commander, Fibre RPC, Multibus, RTT)
barefoc-comms = { version = "0.1", default-features = false, features = ["commander", "rtt"] }

# Optional: Hardware abstraction bridges for your target MCU
barefoc-hal = { version = "0.1", default-features = false, features = ["stm32"] }

# Essential embedded traits
embedded-hal = "1.0"
embedded-io = "0.6"
```

---

## Core Feature Flags

The `barefoc` core crate exposes several feature flags:

| Feature Flag | Default | Description |
|---|---|---|
| `defmt` | Disabled | Enables RTT formatting via the `defmt` logging framework. |
| `boot` | Disabled | Enables `barefoc::boot` mailbox cookies for coordinating Katapult or BareBoot resets. |
| `kinematics` | Disabled | Enables `barefoc::kinematics` (transmissions, leadscrews, 1-DOF/2-DOF joints, backlash compensation). |
| `std` | Disabled | Enables standard library support for host testing, simulations, and unit tests (`cargo test`). |
| `stm32f4` | Disabled | STM32F4 hardware timer and peripheral bridges. |
| `stm32g4` | Disabled | STM32G4 platform support including hardware CORDIC and internal OPAMP integration. |
| `esp32` | Disabled | ESP32 platform support via `esp-hal`. |

---

## Memory & Runtime Setup

Because BareFOC is `#![no_std]`, ensure your firmware provides an architecture-appropriate runtime and panic handler:

```rust,no_run
#![no_std]
#![no_main]

use panic_probe as _; // Formats panics over SWD RTT
// or: use panic_halt as _;
```

### Stack & Memory Footprint

- **Stack Allocation**: All motor models, PID controllers, and filter structs are statically sized. A complete `BLDCMotor` with cascaded PID, low-pass filter, and state variables occupies less than 600 bytes of memory.
- **Heap Allocation**: Zero. BareFOC never calls dynamic memory allocators (`alloc`, `Box`, `Vec`).
