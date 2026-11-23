---
layout: default
title: "UART Break & Stream"
nav_order: 3
parent: "Protocols & Transports"
grand_parent: "BareBoot"
permalink: /bareboot/protocols/uart/
description: "UART serial bootloader protocol: in-band magic tokens, framed break commands, and streaming flash transfers"
---

# UART Serial Protocol & Break Entry

The UART transport provides point-to-point firmware update capabilities over asynchronous serial interfaces (RS-232, RS-485, TTL logic levels, and USB-to-UART bridge ICs such as FTDI, CH340, CP2102, or WCH-LinkE).

---

## 1. Serial Bootloader Entry Triggers

To transition a device running an active application into BareBoot over UART, the host can issue one of three triggers:

### Method A: Katapult In-Band Magic Token String
The host emits the standard Katapult ASCII sync string:
```text
~ \x1c Request Serial Bootloader!! ~
```
The application serial parser intercepts the sequence, writes the cross-reset mailbox token `0x5984E3FA6CA1589B` to RAM, and reboots.

### Method B: Serial Line Break Condition
The host asserts a continuous UART Break (holding the TX line at logic LOW for at least 10 bit-times, exceeding a normal character frame length):
```python
import serial, time
ser = serial.Serial('/dev/ttyUSB0', baudrate=250000)
ser.send_break(duration=0.25) # Assert 250 ms break
```
Microcontrollers with hardware LIN/Break detection (e.g. STM32 USART `LBD` or framing error flags) trigger an interrupt immediately upon break detection.

### Method C: Single-Byte Console Escape Trigger
In development profiles (such as the Miniata USART1 console @ 250,000 baud), the serial listener checks incoming bytes for `0xF0` or character `'r'`:
```rust
//! Miniata in-app UART receiver snippet
pub fn process_uart_byte(byte: u8) {
    if byte == 0xF0 || byte == b'r' {
        unsafe {
            reboot_to_bareboot();
        }
    }
}
```

---

## 2. Serial Wire Framing & Escaping

Because UART is a raw byte stream without hardware packet delimitation, Katapult and BareBoot wrap frames using SLIP-style byte framing:

- **Frame Sync**: `0x7E` (Tilde `~`) marks the beginning and end of each packet.
- **Escape Character**: `0x7D`
  - Byte `0x7E` in payload is encoded as `[0x7D, 0x5E]`.
  - Byte `0x7D` in payload is encoded as `[0x7D, 0x5D]`.

```mermaid
sequenceDiagram
    autonumber
    actor Host as Flashtool Host
    participant MCU as Target UART Peripheral
    participant BL as BareBoot Bootloader

    Host->>MCU: Assert Serial Break or Send In-Band Token
    MCU->>MCU: Write REQUEST_BOOTLOADER & Reset
    Note over MCU,BL: Reset into BareBoot
    Host->>BL: [0x7E, 0x11, CRC16, 0x7E] (CONNECT)
    BL-->>Host: [0x7E, 0x01, ProtoVer, Capabilities, CRC16, 0x7E] (ACK)
    loop Flashing Blocks
        Host->>BL: [0x7E, 0x12, Addr, Len, Data, CRC16, 0x7E] (SEND_BLOCK)
        BL-->>Host: [0x7E, 0x01, CRC16, 0x7E] (ACK)
    end
    Host->>BL: [0x7E, 0x15, CRC16, 0x7E] (COMPLETE)
    BL-->>Host: [0x7E, 0x01, CRC16, 0x7E] (ACK)
```
