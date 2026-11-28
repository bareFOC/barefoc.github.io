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

## 2. Responsive Top Bar & Navigation

`bareConfig` uses a minimal top navigation bar that adapts cleanly to tablets, small laptops, and wide desktops:

- **Home**: Auto-detected hardware connections, **New Board** modal (choose **BareFOC (`barefoc.toml`)** or **BareBOOT (`bareboot.toml`)**), and a fluid 1–3 column grid of board presets fetched from `bareFOC/examples/*/barefoc.toml` (with **Sync from GitHub**).
- **Setup**: Automatically detects **BareFOC Setup** vs **BareBOOT Setup** from the active TOML and shares one **Active Target Header** across 3 unified tabs:
  1. **Overview & Bus**: Hardware pinout, live `parse_barefoc_toml` / `parse_bareboot_toml` validation, auto-populated host bus parameters, and **Apply & Connect**.
  2. **Flash**: Unified **BareFOC App** and **BareBoot / Katapult Bootloader** deployment with a single programmer/bus selector (`probe-rs SWD`, `UART`, `I2C/I3C`, `CAN`), shared **Preserve Calibration (NVM)** toggle, and shared log console.
  3. **TOML Editor**: On-disk / browser TOML editor whose edits sync immediately back to **Overview & Bus** and **Flash**.
- **Workbench**: Appears **only** when connected to a verified BareFOC firmware node (`model_number != 0`), providing **Calibration**, **Oscilloscope**, **Loop Tuning**, **Vector Scope**, **Control Table**, and **Bus Sniffer**.

On viewports narrower than `900px` (down to `480x560`), left sidebars automatically switch to horizontal pill tabs stacked above the content with touch-friendly `34–38px` controls.

The top-right utility bar provides a **GitHub** link button and an icon-only **Theme** toggle cycling **System** (default) -> **Dark (Forest)** -> **Light (Fantasy)**.

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

## 4. Connecting an AI Assistant (MCP — Desktop Only)

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
