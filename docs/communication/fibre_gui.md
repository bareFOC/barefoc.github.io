---
layout: default
title: "Fibre RPC & ODrive GUI"
nav_order: 2
parent: "Communication & GUI"
permalink: /communication/fibre_gui/
---

# Fibre RPC & ODrive GUI (`barefoc-fibre`)

`barefoc-fibre` implements a zero-allocation, `#![no_std]` server for the **Fibre RPC protocol**, allowing any BareFOC device to connect natively to the official [ODrive Web GUI](https://gui.odriverobotics.com) and the `odrivetool` Python CLI.

---

## Why Fibre?

Traditional serial consoles require custom desktop GUI applications for plotting curves. The ODrive Web GUI runs directly in modern Chromium-based browsers via WebUSB or WebSerial, providing:
- Real-time oscilloscope plotting of position, velocity, and phase currents.
- Interactive parameter sliders for PID tuning with immediate visual feedback.
- Automated system identification and step-response profiling.

```
┌─────────────────────────────────┐
│ Browser: ODrive Web GUI         │
│ (https://gui.odriverobotics.com)│
└────────────────┬────────────────┘
                 │ WebSerial / WebUSB / TCP
                 ▼
┌─────────────────────────────────┐
│ Fibre Protocol Engine           │
│ - Packet Framing & CRC-16       │
│ - Endpoint 0 JSON Schema        │
│ - Variable Read / Write / Stream│
└────────────────┬────────────────┘
                 │
                 ▼
┌─────────────────────────────────┐
│ BareFOC Motor Controller State  │
└─────────────────────────────────┘
```

---

## Endpoint 0 JSON Schema

Upon initial connection, the client requests **Endpoint 0**. `barefoc-fibre` returns a compressed JSON schema describing every accessible variable, function, and diagnostic flag exposed by the firmware:

```json
[
  {"name": "vbus_voltage", "id": 1, "type": "float", "access": "r"},
  {"name": "axis0", "members": [
    {"name": "motor", "members": [
      {"name": "current_control", "members": [
        {"name": "Iq_setpoint", "id": 2, "type": "float", "access": "rw"},
        {"name": "Iq_measured", "id": 3, "type": "float", "access": "r"}
      ]}
    ]},
    {"name": "controller", "members": [
      {"name": "vel_setpoint", "id": 4, "type": "float", "access": "rw"},
      {"name": "vel_gain", "id": 5, "type": "float", "access": "rw"}
    ]}
  ]}
]
```

---

## Rust Integration Example

```rust,no_run
use barefoc_fibre::FibreServer;
use embedded_io::{Read, Write};

pub fn handle_fibre_packets<IO: Read + Write>(io: &mut IO, server: &mut FibreServer) {
    let mut rx_buf = [0u8; 128];
    let mut tx_buf = [0u8; 128];

    // Decode incoming binary packet, execute endpoint read/write, transmit response
    server.poll(io, &mut rx_buf, &mut tx_buf);
}
```
