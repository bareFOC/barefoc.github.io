---
layout: default
title: "Minimal Working Example"
nav_order: 2
parent: "Getting Started"
permalink: /getting_started/quickstart/
---

# Minimal Working Example

This guide demonstrates setting up a complete closed-loop velocity control application using a Brushless DC motor, an absolute rotary encoder, and a 3-PWM gate driver.

---

## Architecture Overview

A BareFOC application connects three primary hardware components via Rust traits:
1. **`Sensor`**: Measures mechanical shaft angle ($\theta_m$) and calculates shaft velocity ($\omega_m$).
2. **`BLDCDriver`**: Controls the power inverter, applying PWM duty cycles to the motor phases.
3. **`BLDCMotor`**: Executes the mathematical FOC pipeline, estimating electrical angle ($\theta_e$) and computing phase voltages via Space Vector PWM.

---

## Complete Application Code

```rust,no_run
#![no_std]
#![no_main]

use barefoc::motor::bldc::BLDCMotor;
use barefoc::motor::enums::MotionControlType;
use barefoc::motor::base::FOCMotor;
use barefoc::traits::driver::BLDCDriver;
use barefoc::traits::sensor::Sensor;
use barefoc::traits::Clock;

/// Main motor control task
pub fn motor_task<D, S, C>(driver: &mut D, sensor: &mut S, clock: &C) -> !
where
    D: BLDCDriver,
    S: Sensor,
    C: Clock,
{
    // 1. Instantiate motor with pole pair count (e.g. 7 for a 12N14P motor)
    let mut motor = BLDCMotor::new(7);

    // 2. Specify electrical parameters
    motor.voltage_power_supply = 12.0; // Inverter bus voltage (Volts)
    motor.voltage_limit = 4.0;        // Voltage saturation limit (Volts)
    motor.velocity_limit = 120.0;     // Maximum velocity (rad/s)
    motor.phase_resistance = 0.55;    // Phase-to-neutral resistance (Ohms)
    motor.kv_rating = 4300.0;         // Motor KV rating (RPM/V)

    // 3. Configure control loop & filters
    motor.controller = MotionControlType::velocity;
    motor.PID_velocity.P = 0.08;
    motor.PID_velocity.I = 1.2;
    motor.PID_velocity.D = 0.0;
    motor.PID_velocity.output_ramp = 1000.0; // Limit acceleration (V/s)
    motor.LPF_velocity.Tf = 0.005;           // Velocity filter time constant (5 ms)

    // 4. Initialize hardware abstractions
    motor.init(driver, sensor);

    // 5. Align electrical angle with sensor zero (runs electrical alignment routine)
    motor.init_foc(driver, sensor, clock);

    // 6. Set velocity setpoint (rad/s)
    let mut target_velocity = 40.0; // ~382 RPM

    loop {
        // High-frequency FOC commutation loop:
        // Reads sensor, estimates electrical angle, applies Park/SVPWM transforms
        motor.loop_foc(driver, sensor, clock);

        // Outer motion control loop:
        // Updates velocity PID and calculates target voltage Vq
        motor.move_motor(driver, sensor, clock, target_velocity);
    }
}
```

---

## Step-by-Step Execution Sequence

### Phase 1: Allocation and Configuration
`BLDCMotor::new(pole_pairs)` sets up the internal PID loops, lowpass filters, and voltage limits. Setting `phase_resistance` and `kv_rating` allows the motor to compensate for back-EMF ($e = \omega_e \cdot \lambda_{pm}$) during high-speed rotation.

### Phase 2: Hardware Driver Link (`motor.init`)
`motor.init(driver, sensor)` initializes the gate driver and verifies sensor connectivity. It enables the gate driver output stages and clamps the initial commanded phase voltages to zero.

### Phase 3: Alignment Routine (`motor.init_foc`)
FOC requires precise knowledge of the rotor magnet position relative to the stator electrical coils. During `init_foc()`, BareFOC:
1. Applies an electrical alignment voltage vector at $\theta_e = 0$.
2. Waits for rotor mechanical settling.
3. Samples the sensor to determine `sensor_offset`.
4. Steps the vector forward to detect motor direction (`CW` vs `CCW`).
5. Persists the calibration offsets into internal state, eliminating the need to physically align the sensor magnet.

### Phase 4: Real-Time Control Loop
In the infinite runtime loop:
- `loop_foc()` must be called as fast as possible (ideally $\ge 10\text{ kHz}$) to ensure smooth magnetic flux rotation.
- `move_motor()` executes the motion PID controller, updating target voltages according to the current velocity error.
