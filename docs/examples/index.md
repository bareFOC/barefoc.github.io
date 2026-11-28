---
layout: default
title: "Examples & Reference Designs"
nav_order: 13
has_children: true
permalink: /examples/
---

# Examples & Reference Designs

BareFOC uses a **TOML-only board architecture**: each board in `bareFOC/examples/` contains only a declarative `barefoc.toml` manifest, and the shared `bareFOC/` firmware runner crate synthesizes `memory.x`, `$OUT_DIR/bsp_generated.rs`, and compile-time `--cfg` flags directly from `BAREFOC_BOARD`.

---

## Available Board Definitions

1. [**MicroSpora STM32G431 Reference Design**](/examples/microspora/) *(Officially Recommended Hardware)*:
   Canonical BLDC FOC reference implementation (`examples/microspora/barefoc.toml`) for the officially recommended BareFOC & SimpleFOC hardware ([OSHWLab](https://oshwlab.com/rambros/nano-8316-motor-driver) | [Available at Makerfabs](https://www.makerfabs.com/simplefoc-microspora.html)), pairing an `STM32G431CBU6` (`170 MHz`) with a `DRV8316` 6-PWM + SPI gate driver, `MT6701` 14-bit SSI magnetic encoder, `FDCAN1`, and `VSENS` bus voltage monitoring.

2. [**FishyServo STM32H5 Reference Design**](/examples/fishyservo/):
   Canonical 4-axis multi-topology closed-loop DC servo implementation (`examples/fishyservo/barefoc.toml`, STM32H503KBU6), featuring TMAG5273 3D Hall-effect sensing, dual DRV8411A H-bridges, MultiBus I3C2 target, ASCII Commander, and `H5FlashStorage` v2 persistence.

3. [**Miniata STM32H5 BLDC FOC Smart Servo**](/examples/miniata/):
   Canonical STM32H5 BLDC FOC smart servo reference implementation (`examples/miniata/barefoc.toml`, STM32H503KBU6), featuring DRV8311 3x PWM actuation, dual MT6701 SSI magnetic encoders, cascaded `Actuator1DOF` joint control with planetary gear backlash compensation, MultiBus I2C target, SEGGER RTT Commander, and Flash NVM calibration persistence.

4. **Software-in-the-Loop Simulator (`barefoc-sim`)**:
   Host-side motor simulation connecting to the ODrive GUI over Fibre TCP for tuning FOC parameters without physical hardware.

---

## Building Any Board from `barefoc.toml`

```sh
cd bareFOC

# FishyServo (STM32H503KB + BareBoot)
BAREFOC_BOARD=examples/fishyservo/barefoc.toml cargo build --release --target thumbv8m.main-none-eabihf --features stm32h5,bootloader

# MicroSpora (STM32G431CB)
BAREFOC_BOARD=examples/microspora/barefoc.toml cargo build --release --target thumbv7em-none-eabihf --features stm32g4

# Miniata (STM32H503KB + Katapult/BareBoot)
BAREFOC_BOARD=examples/miniata/barefoc.toml cargo build --release --target thumbv8m.main-none-eabihf --features stm32h5,bootloader
```
