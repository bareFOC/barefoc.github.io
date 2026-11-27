---
layout: default
title: "Configuration Reference"
nav_order: 6
parent: "BareBoot"
permalink: /bareboot/config_reference/
description: "Klipper-style TOML configuration reference for bareboot.toml and bareboot-engine TargetConfig"
---

# BareBoot Configuration Reference

BareBoot supports declarative TOML configuration in two forms:

1. **Standalone `bareboot.toml` (`configs/bareboot-*.toml`)**: Flat board definition parsed by `bareConfig` (`BarebootTomlConfig`) and `bareboot-engine`.
2. **Engine `TargetConfig` (`bareboot.toml`)**: Structured `[runtime_defaults]` and `[memory]` sections used by `bareboot-engine` for builds and safe-pin synthesis.

*(When a board already defines `[bootloader]` inside a BareFOC `barefoc.toml`, `bareConfig` derives the BareBoot configuration from `barefoc.toml` automatically.)*

---

## Standalone `bareboot.toml`

Used by `configs/bareboot.toml` and `configs/bareboot-stm32h503.toml`:

```toml
target = "stm32h503"
#   Target family / profile name (e.g., "stm32h503", "ch32v203", "stm32f1"). Required.
mcu = "stm32h503kb"
#   Specific MCU part number. Optional.
system_clock_hz = 64000000
#   Bootloader core clock frequency in Hz. Default: 64000000.
flash_size = 131072
#   Total internal flash size in bytes. Default: 0 (auto/target default).
ram_size = 32768
#   Total SRAM size in bytes. Default: 0 (auto/target default).
app_base = 0x08004000
#   Application slot start address (e.g., 0x08004000 for 16 KiB bootloader slot).
#   Optional.
boot_window_ms = 250
#   Bootloader wait window in milliseconds before jumping to app. Default: 250.
transports = ["i3c"]
#   Enabled bootloader transports (e.g., ["i3c"], ["can", "i2c"], ["uart"]).
#transport = "i3c"
#   Single active transport override ("can", "i2c", "i3c", "uart", "usb"). Optional.
#baud_rate = 250000
#   UART baud rate when UART transport is enabled. Optional.
#can_bitrate = 1000000
#   CAN / CAN-FD bitrate in bps when CAN transport is enabled. Optional.
i2c_target_address = 0x40
#   7-bit I2C/I3C address (or proposed ENTDAA address). Optional.
```

---

## Engine `TargetConfig` Format (`bareboot-engine`)

### Top-Level Fields

```toml
target = "stm32h503"
#   Target identifier (must be non-empty). Required.
transport = "i2c"
#   Primary transport: "can", "i2c", "uart", "usb", or "multi". Default: "can".
```

---

### `[runtime_defaults]`

```toml
[runtime_defaults]
boot_window_ms = 250
#   Startup wait window before auto-booting the application (ms).
#node_id = 1
#   Optional default CAN/bus node ID.
#i2c_address = 0x40
#   Optional 7-bit I2C/I3C target address.
#uart_baud = 250000
#   Optional UART baud rate.
preserve_nvm = true
#   Whether flashing preserves the application NVM sectors. Default: true.
initial_safe_pins = ["!PA9", "PB1", "^PB12"]
#   Klipper-style pin states applied immediately at bootloader startup before
#   any bus wait window:
#     "PA9"   -> drive high
#     "!PA9"  -> drive low (e.g., keep gate driver nSLEEP / enable inactive)
#     "^PB12" -> input with pull-up
```

---

### `[memory]`

Optional memory map overrides for `bareboot-engine` (omitted fields fall back to target defaults):

```toml
[memory]
#flash_start = 0x08000000
#   Base flash address.
#flash_size = 131072
#   Total flash size in bytes.
#bootloader_size = 16384
#   Reserved bootloader slot size in bytes (e.g., 16384 = 16 KiB).
#app_base = 0x08004000
#   Application start address.
#app_end = 0x0801A000
#   Exclusive end of application flash slot (before CommitStore / NVM).
#nvm_base = 0x0801C000
#   Start of persistent NVM storage sectors.
#page_size = 8192
#   Flash erase page/sector size in bytes.
```
