---
layout: default
title: "Brushed DC Motors"
nav_order: 4
parent: "Actuators & Motors"
permalink: /actuators/dc_motors/
---

# Brushed DC Motors

BareFOC extends its motion control loop architecture to brushed DC motors via the `barefoc-drivers` crate.

---

## Brushed DC Architecture

While DC motors mechanically commutate via brushes and commutator segments, closed-loop position and velocity control still require deterministic PID loops, voltage clamping, and current sensing:

- **`DCDriver2PWM`**: Hardware driver interface controlling an H-bridge via two PWM channels (Forward/Reverse) or PWM + Direction.
- **`DCMotor`**: Motion controller struct providing closed-loop velocity and angle control using standard BareFOC `PIDController` and `LowPassFilter`.
- **`CurrentMirrorSense`**: Lossless current sensing using integrated current mirror outputs found in modern smart H-bridge ICs.

---

## Example: DC Servo Position Control

```rust,no_run
use barefoc_drivers::dc::driver_2pwm::DCDriver2PWM;
use barefoc_drivers::dc::motor::DCMotor;
use barefoc::motor::enums::MotionControlType;

// 1. Initialize 2-PWM H-Bridge driver
let mut driver = DCDriver2PWM::new(pwm_pin_a, pwm_pin_b);
driver.voltage_power_supply = 12.0;
driver.voltage_limit = 10.0;
driver.init();

// 2. Initialize DC Motor controller
let mut dc_motor = DCMotor::new();
dc_motor.controller = MotionControlType::angle;
dc_motor.P_angle.P = 15.0;
dc_motor.PID_velocity.P = 0.2;
dc_motor.PID_velocity.I = 2.0;

// 3. Execution loop
loop {
    dc_motor.move_motor(&mut driver, &mut sensor, &clock, target_angle_rad);
}
```
