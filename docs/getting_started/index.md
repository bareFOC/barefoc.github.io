---
layout: default
title: "Getting Started"
nav_order: 3
has_children: true
permalink: /getting_started/
---

# Getting Started with BareFOC

BareFOC provides both a **TOML-driven firmware runner (`bareFOC`)** for zero-code board bring-up and a **`#![no_std]` Rust FOC library (`barefoc-core`, crate name `barefoc`)** for custom firmware. It targets ARM Cortex-M (STM32), RISC-V (CH32), and Xtensa/ESP32 architectures.

### Two Ways to Use BareFOC

1. **TOML-Driven Firmware Runner (`bareFOC`)**: Define your board in `barefoc.toml` and build with `BAREFOC_BOARD=...`. At compile time, `bareFOC/build.rs` runs `bareConfig` codegen to synthesize `memory.x`, `$OUT_DIR/bsp_generated.rs`, and capability `--cfg` flags, then dispatches to generic capability-driven runners in `bareFOC/src/runners/` with zero per-board `.rs` files.
2. **Core Library (`barefoc-core` / `barefoc` crate)**: Import `barefoc` alongside `barefoc-drivers`, `barefoc-comms`, and `barefoc-hal` in your own Rust firmware crate.

---

## Prerequisites

To use BareFOC, ensure your development environment includes:

1. **Rust Toolchain**:
   A modern stable Rust toolchain (Rust 1.75+ or newer) with the appropriate cross-compilation target:
   ```bash
   # For ARM Cortex-M4F / Cortex-M7F (e.g., STM32F4, STM32G4, STM32H7)
   rustup target add thumbv7em-none-eabihf

   # For ARM Cortex-M33F (e.g., STM32H5)
   rustup target add thumbv8m.main-none-eabihf

   # For RISC-V (e.g., WCH CH32V307)
   rustup target add riscv32imac-unknown-none-elf
   ```

2. **Probe Flashing & Debugging Tools**:
   `probe-rs` or `cargo-flash` for flashing and inspecting RTT debug output:
   ```bash
   cargo install probe-rs-tools
   ```

---

## Getting Started Sections

- [**Cargo Configuration & Features**](/getting_started/cargo_setup/): Setting up `Cargo.toml`, choosing feature flags, linking hardware platforms, and memory layout.
- [**Minimal Working Example**](/getting_started/quickstart/): Step-by-step walkthrough of instantiating drivers, configuring sensors, and running velocity control loops.
