---
layout: default
title: "Target MCUs"
nav_order: 2
parent: "BareBoot"
has_children: true
permalink: /bareboot/targets/
description: "Hardware targets, memory layout partitions, vector offsets, and flashing contracts"
---

# Target Microcontroller Guides

BareBoot provides optimized, hardware-specific bootloader builds tailored to specific silicon architectures and embedded bus controllers.

---

## Supported Target Platforms

| Target MCU | Architecture | Flash Size | Bootloader Slot | User App Base | Primary Bus Transports |
|---|---|---|---|---|---|
| [**STM32H503**](/bareboot/targets/stm32h5/) | ARM Cortex-M33 (250 MHz) | 128 KB (16 sectors x 8 KB) | `0x08000000` (8 KB) | `0x08002000` (Sector 1) | I2C, MIPI I3C, UART |
| [**RP2350 / RP2040**](/bareboot/targets/rp2350/) | Dual ARM Cortex-M33 / Hazard3 | 2 MB – 16 MB External XIP | `0x10000000` (288 KB) | `0x10048000` (App Slot) | USB CDC, UART, SWD Mailbox |
| **STM32F103** | ARM Cortex-M3 (72 MHz) | 64 KB – 128 KB (1 KB/2 KB pages) | `0x08000000` (4 KB – 8 KB) | `0x08002000` | Classic bxCAN, UART |
| **STM32G0B1** | ARM Cortex-M0+ (64 MHz) | 128 KB – 512 KB (2 KB pages) | `0x08000000` (8 KB) | `0x08002000` | FDCAN, UART, USB |
| **CH32V203** | QingKe RISC-V (144 MHz) | 64 KB (Page size: 256 bytes) | `0x08000000` (4 KB) | `0x08001000` | USB 2.0 FS, UART, CAN |

---

## Architecture Selection Principles

Each target platform guide details:
1. **Memory Partition Layout**: Visual flash and SRAM layouts with exact sector, bank, and offset boundaries.
2. **Interrupt Vector Relocation**: Configuration of `SCB->VTOR` (Cortex-M) or `mtvec` (RISC-V) for clean application startup.
3. **Cross-Reset Cookie derived from `bl_vectors[0]`**: Dynamic resolution of the 64-bit boot request signature.
4. **Persistent Configuration Protection**: Dedicated storage sectors isolated from application flash writes (e.g. dual ping-pong config journals).
5. **Peripheral Pin Multiplexing**: Exact GPIO and alternate function assignments for bootloader communication ports.
