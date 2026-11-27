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

1. [**MicroSpora STM32G431 Reference Design**](/examples/microspora/) *(Officially Recommended Hardware)*:
   Canonical BLDC FOC reference implementation for the officially recommended BareFOC & SimpleFOC hardware ([OSHWLab](https://oshwlab.com/rambros/nano-8316-motor-driver) | [Available at Makerfabs](https://www.makerfabs.com/simplefoc-microspora.html)), pairing an `STM32G431CBU6` (`170 MHz`) with a `DRV8316` 6-PWM + SPI gate driver, `MT6701` 14-bit SSI magnetic encoder, `FDCAN1`, and `VSENS` bus voltage monitoring.

2. [**FishyServo STM32H5 Reference Design**](/examples/fishyservo/):
   Canonical 4-axis multi-topology closed-loop DC servo implementation for the FishyServo platform (STM32H503KBU6), featuring TMAG5273 3D Hall-effect sensing, dual DRV8411A H-bridges, MultiBus I2C target, ASCII Commander, and flash configuration persistence.

3. [**Miniata STM32H5 BLDC FOC Smart Servo**](/examples/miniata/):
   Canonical STM32H5 BLDC FOC smart servo reference implementation for the Miniata robotic joint platform (STM32H503KBU6), featuring DRV8311 3x PWM actuation, dual MT6701 SSI magnetic encoders, cascaded `Actuator1DOF` joint control with planetary gear backlash compensation, MultiBus I2C target, SEGGER RTT Commander, and Flash NVM calibration persistence.

4. **STM32F4 BLDC Velocity (`examples/stm32f4_bldc_velocity`)**:
   Complementary 6-PWM velocity closed-loop control on STM32F4 microcontrollers using `embassy-stm32` and hardware quadrature encoder inputs.

5. **Software-in-the-Loop Simulator (`barefoc-sim`)**:
   Host-side motor simulation connecting to the ODrive GUI over Fibre TCP for tuning FOC parameters without physical hardware.
