---
layout: default
title: "Simulation & Testing"
nav_order: 9
has_children: true
permalink: /simulation/
---

# Simulation & Testing (`barefoc-sim`)

`barefoc-sim` provides a pure-Rust, host-side Software-in-the-Loop (SIL) simulation environment for the BareFOC ecosystem.

---

## Why SIL Simulation?

Developing motor control firmware directly on physical microcontrollers carries real-world risks:
- Unstable PID gains or misconfigured electrical angles can trigger catastrophic overcurrent, smoking driver FETs or demagnetizing rotor poles.
- Hardware flash-test-debug cycles are slow and difficult to automate in continuous integration (CI) pipelines.

`barefoc-sim` runs the actual BareFOC library code on the developer's workstation or CI runner, modeling electromagnetic and mechanical physics in real time.

---

## Simulation Documentation Sections

1. [**Physics Engine & ODE Solver**](/simulation/physics_engine/):
   Mathematical modeling of electrical $d$-$q$ stator differential equations, rotor inertia, back-EMF, viscous friction, and sub-stepped numerical integration ($\Delta t \le 1\,\mu\text{s}$).

2. [**Automated Host CI Tests**](/simulation/sil_testing/):
   Headless test automation via `cargo test`. Verifying step responses, settling times, and velocity tracking errors deterministically.

3. [**Live Tuning Bridge**](/simulation/live_tuning/):
   Running a local TCP server providing Fibre RPC to connect simulated motors directly to the ODrive Web GUI for interactive tuning without hardware.
