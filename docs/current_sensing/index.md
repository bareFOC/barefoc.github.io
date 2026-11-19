---
layout: default
title: "Current Sensing"
nav_order: 7
has_children: true
permalink: /current_sensing/
---

# Current Sensing

True torque control and high-bandwidth closed-loop Field Oriented Control require measuring the electrical phase currents flowing through the motor windings.

---

## Current Sensing Documentation Sections

1. [**The `CurrentSense` Trait**](/current_sensing/current_sense_trait/):
   The central interface for phase current measurement, zero-current calibration, and $(I_\alpha, I_\beta)$ transformation.

2. [**Inline Current Sensing**](/current_sensing/inline/):
   Series shunt resistors placed in line with motor phase outputs. Continuous sampling without PWM synchronization constraints.

3. [**Low-Side Current Sensing**](/current_sensing/lowside/):
   Shunts located between low-side MOSFET sources and ground. Requires precise ADC triggering synchronized to the center of the PWM bottom switching period.

4. [**Current Mirror Sensing**](/current_sensing/current_mirror/):
   Lossless current sensing utilizing internal current mirrors inside integrated smart H-bridge and gate driver silicon (`CurrentMirrorSense`).
