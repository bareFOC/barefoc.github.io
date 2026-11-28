---
layout: default
title: "bareConfig Workbench"
parent: "Tools & Utilities"
nav_order: 1
permalink: /tools/bareconfig/
---

# bareConfig User Guide

`bareConfig` is the desktop and web workbench for flashing, calibrating, tuning, and monitoring BareFOC motor controllers and BareBoot bootloaders.

---

## 1. Launching bareConfig

### Web Browser (GitHub Pages / WebAssembly)
Open [https://barefoc.github.io/bareConfig/](https://barefoc.github.io/bareConfig/) directly in any modern browser.

### Desktop Application
```bash
cd bareFOC/bareConfig
cargo run --release
```

---

## 2. Responsive 3-Tab Interface & Themes

`bareConfig` features a minimal, tablet- and laptop-friendly 3-tab top navigation bar:

- **Home**: Auto-detected hardware connections, workspace mode switch (**BareFOC** / **BareBOOT**), and a fluid 1–3 column grid of board presets fetched from `bareFOC/examples/*/barefoc.toml` (with **Sync from GitHub**).
- **Setup**: Unified board configuration and flashing suite (**Board & Pins**, **Host Comm Bus**, **BareBoot Flasher**, **BareFOC Flasher**, and **TOML Editor**).
- **Workbench**: Real-time motor commissioning (**Calibration**, **Oscilloscope**, **Loop Tuning**, **Vector Scope**, **Control Table**, and **Bus Sniffer**) with a wrapping hero telemetry bar, **Arm/Disarm**, and **Emergency Stop**.

On viewports narrower than `900px` (tablets and small windows down to `480x560`), left sidebars automatically switch to horizontal pill tabs stacked above the content with touch-friendly `34–38px` controls.

Use the header toggle to switch between **DaisyUI Forest** (dark) and **DaisyUI Fantasy** (light) themes.

---

## 3. Connecting to Hardware & Programmers

`bareConfig` auto-detects connected host adapters and debug probes on startup (click **Refresh** in Home to re-scan):

- **USB Serial (UART)**: `/dev/ttyACM*`, `/dev/ttyUSB*` (framed `StreamTransport` at `115200..2000000` baud, default `250000`).
- **I2C Bus**: Linux `i2c-dev` (`/dev/i2c-*`) with 16-bit big-endian register pointers.
- **I3C Bus**: Linux `/dev/i3c-*` devices.
- **CAN Bus**: Linux SocketCAN (`can*`, `vcan*`).
- **SWD Programmer (`probe-rs`)**: Raspberry Pi Debugprobe, ST-Link, WCH-Link, and CMSIS-DAP probes via `probe-rs`.
- **Simulation (Offline Demo)**: Virtual actuator node implementing the canonical 512-byte control table.

---

## 4. On-Disk & Browser TOML Editor

Open the **TOML Editor** from any board card (**Edit TOML**) or from **Setup**:

- **Open / Load**, **Save** (downloads `barefoc.toml` in browser WASM builds), **Save As**, and **Quick-Load Board...** operate on `barefoc.toml` and `bareboot.toml`.
- Validation routes `barefoc.toml` through `parse_barefoc_toml`, automatically inferring MCU flash/RAM layout and PWM timer assignments when `[memory]` or `[timers]` are omitted.
- Exporting live settings reads tuned PID gains and limits from the connected actuator and updates `[defaults]` in `barefoc.toml`.

---

## 5. Connecting an AI Assistant (MCP — Desktop Only)

On desktop builds, `bareConfig` includes a built-in Model Context Protocol (MCP) server accessible from the **MCP** button in the top bar (HTTP/SSE on `127.0.0.1:8765`) or via headless stdio (`bareconfig --mcp`):

```json
{
  "mcpServers": {
    "bareconfig": {
      "url": "http://127.0.0.1:8765/sse",
      "command": "bareconfig",
      "args": ["--mcp"]
    }
  }
}
```
