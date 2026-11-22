---
layout: default
title: "Hardware Unique ID"
nav_order: 6
parent: "Platform Hardware Bridges"
permalink: /platforms/hardware_uid/
---

# Hardware Unique Identifier (`barefoc::traits::uid`)

Compact robotic actuators, motor drives, and distributed motion nodes require hardware-unique serial identifiers for bus addressing (I2C, I3C, CAN, RS485), automated URDF joint binding, inventory tracking, and persistent configuration management.

The `barefoc::traits::uid` module provides a `#![no_std]`, zero-allocation trait abstraction over factory silicon electronic serial numbers across microcontroller architectures.

---

## 1. Core Trait Abstraction

The `HardwareUid` trait standardizes hardware serial numbers across disparate bit lengths:

```rust
pub const MAX_UID_BYTES: usize = 12;
pub const CANONICAL_HEX_LEN: usize = 24;

pub trait HardwareUid {
    /// Return the raw hardware UID bytes, with unused trailing bytes zero-padded.
    fn uid_bytes(&self) -> [u8; MAX_UID_BYTES];

    /// Return the actual number of valid bytes in the hardware UID.
    fn uid_len(&self) -> usize;

    /// Format valid UID bytes into uppercase ASCII hex ('0'-'9', 'A'-'F').
    ///
    /// Executes completely on the stack without heap allocation, returning
    /// the number of ASCII characters written (2 * uid_len()).
    fn format_hex(&self, out: &mut [u8; CANONICAL_HEX_LEN]) -> usize;

    /// Derive 6-byte Katapult CAN UUID using FastHash64 with seed 0xA16231A7.
    ///
    /// Matches Katapult CAN discovery format (flashtool.py -u <uuid>).
    fn katapult_uuid(&self) -> [u8; 6];

    /// Derive 48-bit MIPI I3C Provisional ID.
    ///
    /// Format: vendor ID [47:33], vendor-fixed flag [32], model ID [31:16],
    /// instance ID [15:12], and lower silicon UID bits [11:0].
    fn i3c_provisional_id(&self, vendor_id: u16, model_id: u16, instance_id: u8) -> [u8; 6];
}
```

### Zero-Heap Stack Formatter Guarantee

The default implementation of `format_hex` formats bytes directly into caller-provided stack storage:

```rust
let mut hex_buf = [0u8; CANONICAL_HEX_LEN];
let char_count = uid.format_hex(&mut hex_buf);
let hex_str = core::str::from_utf8(&hex_buf[..char_count]).unwrap();
```

---

## 2. Katapult CAN UUID & MIPI I3C Provisional ID Derivation

### Katapult CAN UUID
To integrate seamlessly with Katapult flashtool on multidrop CAN networks without manual UUID configuration, `katapult_uuid()` computes a 64-bit FastHash with Katapult seed `0xA16231A7` over the valid silicon UID bytes:

```rust
let uuid = uid.katapult_uuid();
// Returns 6 bytes matching flashtool discovery
```

### MIPI I3C Provisional ID
During Dynamic Address Assignment (ENTDAA) on MIPI I3C busses, nodes publish a 48-bit Provisional ID. The `i3c_provisional_id()` method produces this identifier deterministically:

```rust
let prov_id = uid.i3c_provisional_id(0x0114, 0x0001, 0x02);
// Returns 6 bytes: [vendor_id, vendor_fixed_flag, model_id, instance_id, uid_bits]
```

---

## 2. Microcontroller Architecture Implementations

| MCU Family | HAL Driver | Silicon Source | Raw Bit Width | `uid_len()` | Memory / Register Source |
|---|---|---|---|---|---|
| **STM32** (F4 / G4 / H5) | `Stm32Uid` (`barefoc-stm32`) | Factory Electronic Signature | 96 bits (12 bytes) | 12 | `embassy_stm32::uid::uid()` / MMIO |
| **RP2350 / RP2040** | `Rp2350Uid` (`barefoc-rp`) | OTP Chip UID / Flash Unique ID | 64 bits (8 bytes) | 8 | OTP rows 0x00..0x03 (`0x4013_0000`) |
| **ESP32** (S3 / C3 / C6) | `Esp32Uid` (`barefoc-esp32`) | Factory Base MAC Address | 48 bits (6 bytes) | 6 | eFuse block 0 / `base_mac_address()` |
| **CH32** (CH32V307) | `Ch32Uid` (`barefoc-ch32`) | ESIG Electronic Signature | 96 bits (12 bytes) | 12 | MMIO register `0x1FFF_F7E8` |

---

## 3. Platform Usage Examples

### STM32 (`barefoc-stm32`)

```rust
use barefoc::traits::HardwareUid;
use barefoc_stm32::Stm32Uid;

let uid = Stm32Uid::new();
let mut hex = [0u8; 24];
let len = uid.format_hex(&mut hex);
// Outputs 24 uppercase hexadecimal characters representing 96-bit factory ID
```

### RP2350 (`barefoc-rp`)

```rust
use barefoc::traits::HardwareUid;
use barefoc_rp::Rp2350Uid;

let uid = Rp2350Uid::new();
assert_eq!(uid.uid_len(), 8);
let bytes = uid.uid_bytes(); // Bytes 0..8 valid, bytes 8..12 zero-padded
```

### ESP32 (`barefoc-esp32`)

```rust
use barefoc::traits::HardwareUid;
use barefoc_esp32::Esp32Uid;

let uid = Esp32Uid::new();
assert_eq!(uid.uid_len(), 6);
let mac = uid.mac_bytes(); // 6-byte EUI-48 base MAC
```

### CH32 RISC-V (`barefoc-ch32`)

```rust
use barefoc::traits::HardwareUid;
use barefoc_ch32::Ch32Uid;

let uid = Ch32Uid::new();
assert_eq!(uid.uid_len(), 12);
```

---

## 4. Integration with Bus Identity & Control Table

The `HardwareUid` trait is used by `ControlTable` (EEPROM register `0x04..0x10`) and `barefoc-multibus` to uniquely distinguish and route packets across multi-drop daisy-chained actuators sharing a physical bus.
