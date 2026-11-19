---
layout: default
title: "Platform Hardware Bridges"
nav_order: 10
has_children: true
permalink: /platforms/
---

# Platform Hardware Bridges (`barefoc-hal`)

BareFOC maintains portable core algorithms by delegating silicon-specific peripheral control to hardware bridge crates in `barefoc-hal`.

---

## Supported Microcontroller Architectures

1. [**STM32 Bridge (`barefoc-stm32`)**](/platforms/stm32/):
   Targets ARM Cortex-M microcontrollers (STM32F4, STM32G4, STM32H5) using `embassy-stm32`. Provides complementary 6-PWM generation with dead-time, hardware timer quadrature decoding, STM32G4 CORDIC math acceleration, and internal OPAMP configuration.

2. [**ESP32 Bridge (`barefoc-esp32`)**](/platforms/esp32/):
   Targets Espressif SoCs (ESP32, ESP32-S3, ESP32-C3, ESP32-C6) using `esp-hal`. Covers MCPWM complementary 6-PWM, LEDC 3-PWM fallback with `hpoint` center-alignment, and PCNT pulse counter encoders.

3. [**CH32 RISC-V Bridge (`barefoc-ch32`)**](/platforms/ch32/):
   Targets WCH RISC-V microcontrollers (CH32V307) using `ch32-hal`. Complementary PWM generation, hardware dead-time, and system timer monotonic clocks.

4. [**Bootloader Soft-Reset (`barefoc::boot`)**](/platforms/bootloader/):
   Coordinating soft-resets into Katapult or BareBoot bootloaders using SRAM mailbox cookies without physical reset buttons.
