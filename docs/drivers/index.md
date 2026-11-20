---
layout: default
title: "Drivers & Inverters"
nav_order: 7
has_children: true
permalink: /drivers/
---

# Drivers & Inverters

Power inverters convert low-voltage microcontroller logic signals into high-current switching waveforms capable of driving motor windings. BareFOC supports multiple gate drive architectures.

---

## Driver Documentation Sections

1. [**Driver Traits**](/drivers/driver_traits/):
   The abstraction layer: `FOCDriver`, `BLDCDriver`, `StepperDriver`, PWM frequency configuration, voltage limits, and enable/fault management.

2. [**PWM Gate Drivers**](/drivers/bldc_drivers/):
   Independent 3-PWM drivers (`BLDCDriver3PWM`) and complementary 6-PWM drivers (`BLDCDriver6PWM`) with hardware dead-time and break inputs.

3. [**Smart Gate Drivers (`drv83xx-rs`)**](/drivers/smart_gate_drivers/):
   Hardware-managed smart gate drivers from Texas Instruments: DRV8316 (integrated FETs) and DRV8311 (24-bit/32-bit SPI/tSPI, PWMG, cross-talk decoupling).
