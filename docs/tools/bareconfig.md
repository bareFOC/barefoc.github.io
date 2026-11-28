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

### Desktop Application
```bash
cd bareFOC/bareConfig
cargo run --release
```

### Web Browser (WebAssembly / WebSerial)
```bash
cd bareFOC/bareConfig
trunk serve --release
```

---

## 2. Beginner Mode vs. Advanced Mode

`bareConfig` starts in **Beginner Mode** by default to provide a clean, guided zero-to-spin experience:

- **Beginner Mode**:
  - **Home Hub** shows auto-detected USB adapters and SWD probes alongside board presets discovered dynamically from `bareFOC/examples/*/barefoc.toml` (`miniata`, `fishyservo`, `microspora`, and any custom boards in `./boards/`).
  - **Guided 5-Step Setup** walks through:
    1. **Select Board**: Pick your motor controller preset.
    2. **Connect & Flash**: Choose an auto-detected adapter/probe (or Simulation) and flash firmware.
    3. **Sensor Check**: Rotate the shaft by hand and verify smooth angle readout.
    4. **Align & Test Spin**: Run automatic electrical zero alignment and a safe low-speed test spin.
    5. **Save**: Commit calibration to Sector 15 flash NVM and export tuned settings back to `barefoc.toml`.
  - Every board card also provides an **Edit TOML** shortcut to inspect or edit the board's `barefoc.toml` directly.
- **Advanced Mode** (1-click toggle in the top header):
  - Unlocks the full 5-page **Setup** sidebar (`Board & Pins`, `Host Comm Bus`, `BareBoot Flasher`, `BareFOC Flasher`, `TOML Editor`).
  - Unlocks all 6 **Motor Workbench** workspaces: **Calibration**, **Oscilloscope**, **Loop Tuning** (cascaded PID diagram), **Vector Scope**, **Control Table** (512-byte canonical hex inspector with search), and **Bus Sniffer**.

---

## 3. Connecting to Hardware & Programmers

`bareConfig` auto-detects connected host adapters and debug probes on startup (click **Refresh** in the Home Hub to re-scan):

- **USB Serial (UART)**: `/dev/ttyACM*`, `/dev/ttyUSB*` (framed `StreamTransport` at 115200..2000000 baud, default `250000`).
- **I2C Bus**: Linux `i2c-dev` (`/dev/i2c-*`) with 16-bit big-endian register pointers.
- **I3C Bus**: Linux `/dev/i3c-*` devices.
- **CAN Bus**: Linux SocketCAN (`can*`, `vcan*`).
- **SWD Programmer (`probe-rs`)**: Raspberry Pi Debugprobe, ST-Link, WCH-Link, and CMSIS-DAP probes via `probe-rs`.
- **Simulation (Offline Demo)**: Virtual actuator node implementing the canonical 512-byte control table.

---

## 4. On-Disk TOML Editor & Settings Export

Open the **TOML Editor** from any board card (**Edit TOML**) or from the Advanced Setup sidebar:

- **Open / Load**, **Save**, **Save As**, and **Quick-Load Board...** operate directly on `barefoc.toml` and `bareboot.toml` files on disk.
- Validation routes `barefoc.toml` through `parse_barefoc_toml`, automatically inferring MCU flash/RAM layout and PWM timer assignments when `[memory]` or `[timers]` are omitted.
- Clicking **Save to barefoc.toml** (or **Export Settings to TOML**) reads live PID gains and limits from the connected actuator and updates `[defaults]` in the target `barefoc.toml` file on disk.

---

## 5. Connecting an AI Assistant (MCP)

`bareConfig` includes a built-in Model Context Protocol (MCP) server so AI coding assistants (such as Antigravity, Claude Desktop, or Cursor) can connect over `uart`, `i2c`, `i3c`, `can`, `probe-rs`, or `simulated` transports to read telemetry, tune PIDs, inspect canonical control-table registers, validate `barefoc.toml` via `parse_barefoc_toml`, and flash firmware.

- **With the GUI running**: Click **Copy MCP Config** in the header (connects to `http://127.0.0.1:8765/sse`).
- **Headless CLI mode**: Run `bareconfig --mcp` via stdio.

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
