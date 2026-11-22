---
layout: default
title: "Examples & Reference Designs"
nav_order: 13
has_children: true
permalink: /examples/
---

# Examples & Reference Designs

BareFOC includes production-ready reference implementations illustrating end-to-end integration across microcontrollers, gate drivers, magnetic angle sensors, kinematics assemblies, and communication transports.

---

## Available Reference Implementations

1. [**FishyServo STM32H5 Reference Design**](/examples/fishyservo/):
   Canonical 4-axis multi-topology closed-loop DC servo implementation for the FishyServo platform (STM32H503KBU6), featuring TMAG5273 3D Hall-effect sensing, dual DRV8411A H-bridges, MultiBus I2C target, ASCII Commander, and flash configuration persistence.

2. **STM32F4 BLDC Velocity (`examples/stm32f4_bldc_velocity`)**:
   Complementary 6-PWM velocity closed-loop control on STM32F4 microcontrollers using `embassy-stm32` and hardware quadrature encoder inputs.

3. **Software-in-the-Loop Simulator (`barefoc-sim`)**:
   Host-side motor simulation connecting to the ODrive GUI over Fibre TCP for tuning FOC parameters without physical hardware.
