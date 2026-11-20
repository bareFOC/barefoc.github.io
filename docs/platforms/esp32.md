---
layout: default
title: "ESP32 Bridge"
nav_order: 2
parent: "Platform Hardware Bridges"
permalink: /platforms/esp32/
---

# ESP32 Hardware Bridge (`barefoc-esp32`)

`barefoc-esp32` connects Espressif's `esp-hal` to the BareFOC ecosystem, supporting ESP32, ESP32-S3, ESP32-C3, and ESP32-C6.

---

## 1. Motor Control PWM (`Esp32McpwmDriver`)

Available on ESP32 and ESP32-S3:
- Uses the dedicated **MCPWM (Motor Control Pulse Width Modulator)** peripheral.
- Generates 3 complementary phase pairs with hardware dead-time generators and cycle-by-cycle fault shutdown.
- Up-down center-aligned counter synchronization.

---

## 2. Universal LEDC 3-PWM Driver (`Esp32LedcDriver`)

Available across all ESP32 variants:
- Employs the LEDC hardware timer.
- Configures phase center-alignment using `hpoint` register offsets to simulate center-aligned PWM across standard low-cost channels.

---

## 3. Hardware Pulse Counter (`Esp32PcntSensor`)

Directly drives the ESP32 PCNT (Pulse Counter) unit:
- Glitch filters remove mechanical contact bounce and electrical noise.
- Low-power, interrupt-free pulse counting up to tens of megahertz.

---

## 4. Hardware Silicon Identifier (`Esp32Uid`)

Retrieves the 48-bit factory base MAC address from eFuse, implementing `barefoc::traits::HardwareUid` with zero-allocation canonical hex formatting and trailing zero padding to 12 bytes.
