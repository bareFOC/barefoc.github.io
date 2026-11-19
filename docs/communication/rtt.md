---
layout: default
title: "SEGGER RTT Transport"
nav_order: 4
parent: "Communication & GUI"
permalink: /communication/rtt/
---

# SEGGER RTT Transport (`barefoc-rtt`)

SEGGER Real-Time Transfer (RTT) allows bidirectional communication between a host computer and an embedded target over standard J-Link or CMSIS-DAP/SWD debug probes without consuming physical UART pins or hardware timers.

---

## Operating Principle

RTT operates by placing circular ring buffers directly into target microcontroller SRAM:
- The host probe reads and writes these buffers via non-intrusive background memory access (SWD background memory reads) while the CPU is executing full-speed FOC loops.
- Execution overhead is negligible ($\approx 1\text{ microsecond}$ per transmission), preventing debug logging from disrupting critical 25 kHz FOC timing.

---

## Rust Integration Example

`barefoc-rtt` wraps RTT channels in the standard `embedded_io::Read` and `embedded_io::Write` traits:

```rust,no_run
use barefoc_rtt::RttStream;
use barefoc_commander::Commander;

// Connect to RTT Up Channel 0 (Target to Host) and Down Channel 0 (Host to Target)
let mut stream = RttStream::new();

// Pass directly into Commander or logging utilities
commander.process(&mut stream);
```
