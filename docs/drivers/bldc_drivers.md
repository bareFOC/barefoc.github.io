---
layout: default
title: "PWM Gate Drivers"
nav_order: 2
parent: "Drivers & Inverters"
permalink: /drivers/bldc_drivers/
---

# PWM Gate Drivers

BareFOC provides two primary driver implementations for 3-phase BLDC motors:
1. `BLDCDriver3PWM`
2. `BLDCDriver6PWM`

---

## 1. 3-PWM Driver (`BLDCDriver3PWM`)

The 3-PWM driver uses three independent PWM channels, typically driving integrated half-bridge power stages (such as the L6234, MP6540, or DRV8313) where internal logic automatically generates complementary high-side and low-side switching with built-in dead-time.

```
MCU Pin 1 (PWM A) ───► Inverter Phase A (High/Low)
MCU Pin 2 (PWM B) ───► Inverter Phase B (High/Low)
MCU Pin 3 (PWM C) ───► Inverter Phase C (High/Low)
MCU Pin 4 (Enable)───► Inverter Enable (Optional)
```

### Rust Usage:
```rust,no_run
use barefoc::drivers::bldc_3pwm::BLDCDriver3PWM;
use barefoc::traits::driver::BLDCDriver;

let mut driver = BLDCDriver3PWM::new(pwm_a, pwm_b, pwm_c, Some(enable_pin));
driver.voltage_power_supply = 12.0;
driver.voltage_limit = 10.0;
driver.init();
```

---

## 2. Complementary 6-PWM Driver (`BLDCDriver6PWM`)

For high-power inverters built from discrete MOSFETs or dedicated half-bridge drivers (e.g. DRV8301, FD6288), the microcontroller must independently control both the high-side FET and low-side FET of each phase.

```
Phase A: High-Side PWM (INHA) + Low-Side PWM (INLA)
Phase B: High-Side PWM (INHB) + Low-Side PWM (INLB)
Phase C: High-Side PWM (INHC) + Low-Side PWM (INLC)
```

### Critical Requirements:
- **Hardware Dead-Time Insertion**: When transitioning a phase from high-side to low-side, both transistors must be briefly off ($\approx 50\text{--}500\text{ ns}$) to prevent catastrophic shoot-through current directly from the DC rail to ground.
- **Center-Aligned Switching**: PWM counters count up then down (dual-slope), aligning the center of all pulses to cancel high-frequency ripple harmonics.

In BareFOC, 6-PWM drivers are typically backed by microcontroller hardware timers using `barefoc-hal` (`barefoc-stm32` Advanced Timers TIM1/TIM8 or `barefoc-esp32` MCPWM).
