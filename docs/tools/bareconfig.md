---
layout: default
title: "bareConfig Workbench"
nav_order: 12
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

## 2. Connecting to a Board or Project

When `bareConfig` opens, use the left sidebar in the **Home Hub** to either open a board definition or connect directly to hardware:

1. **Connect to Hardware**:
   - Select your transport (**Simulation**, **UART Serial**, **I2C / I3C Bus**, or **CAN Bus**).
   - Enter the port or bus address and baud rate (for example, `250000` for UART or `1000000` for I3C), then click **Connect Hardware**.
2. **Open or Create a Project**:
   - Click **New BareFOC Project**, **New BareBOOT Target**, or **Open TOML / Directory** to load a `barefoc.toml` board file.
   - Or pick a built-in template (**FishyServo**, **Miniata**, or **Generic Bootloader**) and click **Open Setup**.
3. **Switch Workspace Mode**:
   - Use the bottom-left toggle to switch between **BareFOC Mode** (firmware, calibration, tuning) and **BareBOOT Mode** (bootloader configuration and flashing).

---

## 3. Setting Up & Flashing Firmware

Open the **BareFOC Setup** (or **BareBOOT Setup**) tab to configure silicon pins, bus settings, and flash firmware:

1. **Board & Pin Mapping**:
   - Review the MCU pin assignments for PWM phases, gate sleep/fault, current sense ADC, sensor bus, and host communication.
   - Confirm the safe-pin check reports `[ VALID - initial_safe_pins! OK ]` before flashing.
2. **Host Communication**:
   - Configure UART baud rate, I2C/I3C address and speed, or CAN bitrate and Node ID.
   - See the [Configuration Reference](/config_reference/) for all `barefoc.toml` options.
3. **TOML Editor**:
   - Edit `barefoc.toml` directly with live schema validation, or click **Export Settings to TOML** after tuning to save live gains back to disk.
4. **Flashing Bootloader & Firmware**:
   - Use **BareBoot Flasher** to flash the bootloader to the boot slot.
   - Use **BareFOC Flasher** to compile and flash application firmware.
   - Enable **Preserve Persistent Settings (Sector 15 NVM)** when updating firmware on an already-calibrated board so your motor calibration, Node ID, and PID gains are kept intact.

---

## 4. Calibrating & Tuning a Motor (`Motor Workbench`)

Once connected to a running BareFOC node, the **Motor Workbench** tab appears in the top bar.

### Top Telemetry Bar
Always visible across every workbench tool:
- Live **Rotor Angle** (deg / rad), **Velocity** (RPM / rad/s), **Phase Currents** ($I_q, I_d$), **Bus Voltage** ($V_{bus}$), and **Temperature**.
- **Arm / Disarm** toggle for closed-loop drive.
- **Emergency Stop** button to immediately shut down PWM outputs and coast the motor safely.

### Step-by-Step Commissioning Workflow
1. **Motor Calibration**:
   - Verify sensor readout and magnetic field health.
   - Run **Electrical Zero Alignment** to measure pole pairs, sensor direction, and electrical zero offset.
   - Use the **Jog** slider to confirm smooth bidirectional motion, then click **Save to NVM** to persist calibration to flash.
2. **Closed-Loop Tuning**:
   - Click any gain pill in the cascaded loop diagram (**Position P**, **Velocity PID**, **Current PI**, **LPF $T_f$**) to adjust gains live.
3. **Oscilloscope & Step Benchmark**:
   - Plot real-time position, velocity, and current waveforms at up to 60 FPS.
   - Run an automated **Step-Response Benchmark** to measure rise time, settling time, and overshoot.
4. **Vector & SVPWM Scope**:
   - Inspect the live $(I_d, I_q)$ / $(\alpha, \beta)$ current vector and 6-sector SVPWM duty cycle hexagon.
5. **Diagnostics**:
   - **Control Table Inspector**: Search, read, and write individual registers in the live control table.
   - **Bus Traffic Sniffer**: Capture, pause, and export MultiBus, BareBoot/Katapult, DXL2, and Commander frames.

---

## 5. Connecting an AI Assistant (MCP)

`bareConfig` includes a built-in Model Context Protocol (MCP) server so AI coding assistants (such as Antigravity, Claude Desktop, or Cursor) can read telemetry, tune PIDs, run benchmarks, and flash firmware.

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
