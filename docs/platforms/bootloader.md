---
layout: default
title: "Bootloader Soft-Reset"
nav_order: 4
parent: "Platform Hardware Bridges"
permalink: /platforms/bootloader/
---

# Bootloader Soft-Reset (`barefoc::boot`)

Compact robotics actuators often lack physical reset buttons, tactile switches, or exposed SWD programming headers once mechanically enclosed.

The `barefoc::boot` module coordinates soft-reset requests between application firmware and embedded bootloaders (such as **Katapult** or **BareBoot**) using persistent SRAM mailbox cookies.

---

## The Mailbox Cookie Mechanism

During normal application runtime, when a host issues a firmware flash command over UART, I2C, or USB, the application:

1. **Disarms Inverter Bridges**: Immediately shuts down PWM generation and pulls gate driver sleep pins low (`nSLEEP = LOW`).
2. **Disables Interrupts**: Prevents pending timer or peripheral interrupts from triggering.
3. **Writes Mailbox Signature**: Writes a 64-bit magic cookie value to an uninitialized memory address at the top of SRAM (e.g. `0x2000_7FF8` on STM32H503):
   ```rust
   // Katapult REQUEST_CANBOOT signature
   const KATAPULT_MAGIC: u64 = 0x5984_E3FA_6CA1_589B;
   ```
4. **Triggers System Soft-Reset**: Executes an architecture-level system reset (`NVIC_SystemReset()` on ARM Cortex-M or PFIC reset on RISC-V).

---

## Bootloader Execution on Reset

When the MCU restarts, the bootloader (residing in Flash Sector 0) initializes before the application:
- It inspects the magic cookie address in SRAM.
- If the signature matches, the bootloader cancels its automatic timeout jump to the application, holding in bootloader flashing mode permanently.
- If the signature is absent (power cycle), the bootloader waits for its brief communication window (e.g. 250 ms) before booting Sector 1 application firmware.

---

## Rust Integration Example

```rust,no_run
use barefoc::boot::MailboxCookie;

pub fn reboot_to_bootloader() -> ! {
    // 1. Safe shutdown of gate driver
    // 2. Write signature and trigger reset
    MailboxCookie::write_and_reset(0x2000_7FF8, 0x5984_E3FA_6CA1_589B);
}
```
