---
layout: default
title: "Actuators & Motors"
nav_order: 4
has_children: true
permalink: /actuators/
---

# Actuators & Motors

BareFOC supports multiple electromagnetic actuator topologies through unified motor traits and decoupled state machines.

---

## Supported Actuator Topologies

1. [**BLDC Motors (`BLDCMotor`)**](/actuators/bldc/):
   3-phase permanent magnet synchronous motors (PMSM / BLDC). Covers pole pair definitions, winding resistance, inductance, KV rating, and motion control modes.

2. [**Stepper Motors (`StepperMotor`)**](/actuators/stepper/):
   2-phase bipolar stepper motors driven with true Field Oriented Control for silent operation, zero resonance, and high efficiency.

3. [**Hybrid Steppers (`HybridStepperMotor`)**](/actuators/hybrid_stepper/):
   Dual-regime motors switching between high-speed open-loop microstepping and low-speed closed-loop FOC with stall detection.

4. [**Brushed DC Motors (`DCMotor`)**](/actuators/dc_motors/):
   Single-phase H-bridge brushed DC motors driven via `DCDriver2PWM` using BareFOC's cascaded velocity and position PID loops.

5. [**Kinematics & Transmissions (`barefoc::kinematics`)**](/actuators/kinematics/):
   Mechanical transmission modeling, gear reduction ratios, leadscrews, 1-DOF / 2-DOF joint kinematics, and mechanical backlash compensation.
