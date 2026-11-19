---
layout: default
title: "Automated Host CI Tests"
nav_order: 2
parent: "Simulation & Testing"
permalink: /simulation/sil_testing/
---

# Automated Host CI Tests

`barefoc-sim` enables automated testing of motion control routines in standard Rust host unit tests without microcontrollers or physical motors.

---

## Writing an Automated Step-Response Test

```rust,no_run
use barefoc_sim::{MotorParams, SimClock, SimRunner};
use barefoc::motor::enums::MotionControlType;
use barefoc::motor::base::FOCMotor;

#[test]
fn test_velocity_convergence() {
    let clock = SimClock::new();
    let params = MotorParams::miniata_1104_4300kv();
    
    // Create runner with 40 microsecond FOC loop interval (25 kHz)
    let mut runner = SimRunner::new(&clock, params, 40e-6);

    // Configure controller parameters
    runner.motor.cfg_mut().update_motion_control_type(MotionControlType::velocity);
    runner.motor.cfg_mut().PID_velocity.P = 0.05;
    runner.motor.cfg_mut().PID_velocity.I = 1.0;
    runner.motor.cfg_mut().voltage_limit = 3.0;

    // Run electrical calibration
    runner.init();
    runner.init_foc();

    // Step simulation forward for 0.5 physical seconds commanding 60 rad/s
    runner.step_for(0.5, 60.0);

    // Assert that velocity settled within 1.0 rad/s of target
    let final_velocity = runner.state().shaft_velocity;
    assert!(
        (final_velocity - 60.0).abs() < 1.0,
        "Velocity did not converge: measured {}",
        final_velocity
    );
}
```

---

## Benefits for CI Pipelines

- **Execution Speed**: 10 seconds of simulated motor run time executes in under 50 milliseconds of real CPU time.
- **Deterministic**: Physics and control loops use fixed seeds and pseudo-time, producing identical, reproducible outputs across Linux, macOS, and Windows runners.
- **Regression Prevention**: Quantifies steady-state tracking error, overshoot percentage, and phase margins on every pull request.
