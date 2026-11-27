---
layout: default
title: "Configuration Reference"
nav_order: 6
parent: "BareBoot"
permalink: /bareboot/config_reference/
description: "Unified Klipper-style TOML configuration reference shared by bareboot.toml and barefoc.toml"
---

# BareBoot Configuration Reference

BareBoot and BareFOC share a single unified Klipper-style TOML schema (`[board]`, `[memory]`, `[communication]`, `[bootloader]`) parsed identically by `bareConfig` and `bareboot-engine`:

- **Standalone `bareboot.toml` (`configs/bareboot-*.toml`)**: Contains `[board]`, `[memory]`, `[communication]`, and `[bootloader]`.
- **Unified `barefoc.toml`**: Contains the exact same four sections plus motor/driver/sensor sections. When `[bootloader].initial_safe_pins` is omitted, `bareboot-engine` auto-derives safe startup pins from `[gate_driver]`.

---

## `[board]`

Defines the silicon target and core clock.

```toml
[board]
name = "stm32h503-generic"
#   Optional human-readable board identifier.
mcu = "stm32h503kb"
#   Specific MCU part number (e.g., "stm32h503kbu6", "ch32v203c8t6", "stm32f103c8").
family = "stm32h5"
#   Target MCU family ("stm32h5", "stm32g4", "stm32f1", "ch32v203").
system_clock_hz = 64000000
#   Bootloader core clock frequency in Hz. Default: 64000000.
```

---

## `[memory]` *(optional — auto-derived)*

Flash and SRAM partition boundaries shared identically with `barefoc.toml`:

```toml
[memory]
flash_origin = 0x08000000
#   Base flash address.
flash_size = 131072
#   Total internal flash size in bytes.
ram_origin = 0x20000000
#   Base SRAM address.
ram_size = 32768
#   Total SRAM size in bytes.
bootloader_origin = 0x08000000
#   Bootloader slot start address.
bootloader_size = 16384
#   Reserved bootloader slot size in bytes (e.g., 16384 = 16 KiB).
app_origin = 0x08004000
#   Application slot start address.
app_size = 90112
#   Application slot size in bytes (before CommitStore / NVM).
nvm_origin = 0x0801C000
#   Start of persistent NVM storage sectors.
nvm_size = 16384
#   Persistent NVM size in bytes.
page_size = 8192
#   Flash erase page/sector size in bytes.
```

---

## `[communication]`

Configures bootloader transport(s), bus pins, bitrate, and target address:

```toml
[communication]
transports = ["i3c", "i2c"]
#   Enabled bootloader transports ("i3c", "i2c", "can", "uart", "usb", "multi").
#transport = "i3c"
#   Optional single primary transport override.
#tx_scl_pin = "PB3"
#rx_sda_pin = "PB4"
#can_enable_pin = "PC13"
bitrate = 1000000
#   Bus bitrate in bps (e.g., 1000000 = 1 MHz CAN/I3C, 250000 = UART). Optional.
address = 0x40
#   7-bit I2C/I3C target address (or proposed ENTDAA address). Optional.
```

---

## `[bootloader]`

Configures boot window, NVM preservation, Klipper-style safe startup pins, and soft-reset mailbox:

```toml
[bootloader]
enabled = true
#   Whether the application links behind a bootloader slot. Default: true.
type = "bareboot"
#   Bootloader type: "bareboot" or "katapult". Default: "bareboot".
boot_window_ms = 250
#   Startup wait window before auto-booting the application (ms). Default: 250.
preserve_nvm = true
#   Whether flashing preserves the application NVM sectors. Default: true.
initial_safe_pins = ["!PA9", "PB1", "^PB12"]
#   Klipper-style pin states applied immediately at bootloader startup before
#   any bus wait window:
#     "PA9"   -> drive high
#     "!PA9"  -> drive low (e.g., keep gate driver nSLEEP / enable inactive)
#     "^PB12" -> input with pull-up
#     "~PB1"  -> input with pull-down
mailbox_address = 0x20007FE0
#   SRAM address of the soft-reset / boot-loop mailbox. Optional.
mailbox_magic = 0x5984E3FA6CA1589B
#   64-bit magic signature requesting bootloader entry. Optional.
```
