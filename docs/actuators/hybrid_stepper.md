---
layout: default
title: "Hybrid Steppers"
nav_order: 3
parent: "Actuators & Motors"
permalink: /actuators/hybrid_stepper/
---

# Hybrid Stepper Motors (`HybridStepperMotor`)

The `HybridStepperMotor` struct in `barefoc::motor::hybrid_stepper` implements dual-regime operation, seamlessly transitioning between closed-loop FOC and high-speed microstepping.

---

## Operating Regimes

1. **Closed-Loop FOC Regime (Low to Medium Speeds)**:
   - Uses absolute encoder feedback to maintain an exact 90° load angle.
   - Eliminates acoustic noise and cuts power consumption by up to 75% compared to open-loop drivers.
2. **Open-Loop Transition Regime (Ultra-High Speeds)**:
   - Beyond the maximum bandwidth of the position sensor or ADC sampling, the motor smoothly transitions into open-loop field-weakening microstepping to maximize top speed.

---

## Configuration Example

```rust,no_run
use barefoc::motor::hybrid_stepper::HybridStepperMotor;
use barefoc::motor::enums::MotionControlType;

let mut hybrid = HybridStepperMotor::new(50);

// Configure transition threshold (rad/s)
hybrid.transition_velocity = 80.0; // Transition to open-loop above ~760 RPM
hybrid.holding_current_factor = 0.3; // Reduce current to 30% when stationary

hybrid.init(driver, sensor);
hybrid.init_foc(driver, sensor, clock);
```
