---
layout: default
title: "Live Tuning Bridge"
nav_order: 3
parent: "Simulation & Testing"
permalink: /simulation/live_tuning/
---

# Live Tuning Bridge

`barefoc-sim` includes a live network bridge that exposes the simulated motor over a local TCP socket implementing the Fibre RPC protocol.

---

## Architecture

```
┌─────────────────────────────────┐
│ Browser: ODrive Web GUI         │
│ (https://gui.odriverobotics.com)│
└────────────────┬────────────────┘
                 │ WebRTC / TCP Socket (127.0.0.1:9999)
                 ▼
┌─────────────────────────────────┐
│ barefoc-sim live server         │
│ - Fibre RPC Protocol Handler    │
│ - Continuous Physics Loop       │
│ - Real-Time Telemetry Stream    │
└─────────────────────────────────┘
```

---

## Launching the Tuning Bridge

To launch the simulated motor server locally:

```bash
cargo run --package barefoc-sim --example live_tuning -- --port 9999
```

1. Open Chromium or Chrome and navigate to [https://gui.odriverobotics.com](https://gui.odriverobotics.com).
2. Select **Connect** $\to$ **TCP Socket** $\to$ `127.0.0.1:9999`.
3. The GUI automatically detects the virtual BareFOC motor, loading live oscilloscope traces for shaft velocity, phase voltages, and motor position.
4. Drag PID sliders in the browser to test gains in real time.
