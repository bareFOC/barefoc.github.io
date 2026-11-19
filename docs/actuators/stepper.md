---
layout: default
title: "Stepper Motors"
nav_order: 2
parent: "Actuators & Motors"
permalink: /actuators/stepper/
---

# Stepper Motors (`StepperMotor`)

The `StepperMotor` struct in `barefoc::motor::stepper` controls 2-phase bipolar stepper motors using Field Oriented Control.

---

## Why FOC for Stepper Motors?

Traditional stepper motor drivers use step/direction pulses with open-loop current chopping (microstepping). This produces notable drawbacks:
- Constant full current consumption even when stationary or lightly loaded.
- Severe acoustic resonance and vibration at mid-band frequencies.
- Complete loss of position (stall) if external torque exceeds holding torque.

BareFOC treats a 2-phase stepper motor as a 2-phase synchronous AC machine with 50 pole pairs (for a standard 1.8° stepper):

$$\theta_e = 50 \cdot \theta_m$$

### Key Benefits:
- **True Current-on-Demand**: The motor draws only the current needed to overcome the instantaneous load torque.
- **Silent Operation**: Smooth sinusoidal 2-phase rotating magnetic field eliminates step resonance.
- **Closed-Loop Reliability**: External loads cannot cause missed steps; the motor continues tracking or raises a stall fault.

---

## Configuration Example

```rust,no_run
use barefoc::motor::stepper::StepperMotor;
use barefoc::motor::enums::MotionControlType;
use barefoc::motor::base::FOCMotor;

// 1. Standard 1.8° stepper motor has 50 pole pairs (200 full steps/rev)
let mut stepper = StepperMotor::new(50);

stepper.voltage_power_supply = 24.0; // Inverter supply voltage
stepper.voltage_limit = 12.0;
stepper.velocity_limit = 50.0;       // rad/s

stepper.controller = MotionControlType::angle;
stepper.P_angle.P = 20.0;
stepper.PID_velocity.P = 0.5;
stepper.PID_velocity.I = 5.0;

// Initialize with a 4-PWM or 2-PWM stepper driver and position sensor
stepper.init(driver, sensor);
stepper.init_foc(driver, sensor, clock);

loop {
    stepper.loop_foc(driver, sensor, clock);
    stepper.move_motor(driver, sensor, clock, target_angle_rad);
}
```

---

## Inverter Topologies for Stepper Motors

Steppers use 2 independent H-bridges:
- **`StepperDriver4PWM`**: 4 PWM pins controlling phase A+, A-, B+, B- independently.
- **`StepperDriver2PWM`**: 2 PWM pins (Phase A, Phase B magnitude) with 2 Direction pins.
