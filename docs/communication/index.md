---
layout: default
title: "Communication & GUI"
nav_order: 8
has_children: true
permalink: /communication/
---

# Communication & GUI

The `barefoc-comms` workspace provides `#![no_std]` serial command parsing, memory-mapped bus tables, and binary RPC protocols for live motor tuning and telemetry.

---

## Communication Protocols

1. [**Commander Protocol**](/communication/commander/):
   The ASCII command-line parser implementing the SimpleFOC `commands.h` grammar over any `embedded_io::Read` and `embedded_io::Write` stream.

2. [**Fibre RPC & ODrive GUI**](/communication/fibre_gui/):
   Binary packetized RPC engine natively compatible with the official [ODrive Web GUI](https://gui.odriverobotics.com) and `odrivetool` for real-time oscilloscope plotting and interactive slider tuning.

3. [**Multibus Register Protocol**](/communication/multibus/):
   Descriptor-driven control table framework for address-based multi-drop buses (I2C, SPI, I3C).

4. [**SEGGER RTT Transport**](/communication/rtt/):
   Zero-overhead bidirectional byte-stream transport over SWD debug probes implementing `embedded-io`.
