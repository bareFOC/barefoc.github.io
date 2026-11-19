---
layout: default
title: "Driver Traits"
nav_order: 1
parent: "Drivers & Inverters"
permalink: /drivers/driver_traits/
---

# Driver Traits

BareFOC decouples motor state machines from physical silicon timers via the driver traits defined in `barefoc::traits::driver`.

---

## 1. The `FOCDriver` Base Trait

`FOCDriver` defines the common lifecycle methods shared by all power stages:

```rust
pub trait FOCDriver {
    /// Initialize the driver hardware (configure GPIOs, PWM frequency, timers)
    fn init(&mut self) -> Result<(), ()>;

    /// Enable gate drive outputs
    fn enable(&mut self);

    /// Disable gate drive outputs (float phases or place in high impedance)
    fn disable(&mut self);

    /// Set maximum voltage rail (power supply voltage)
    fn voltage_power_supply(&self) -> f32;

    /// Set allowable voltage ceiling
    fn voltage_limit(&self) -> f32;
}
```

---

## 2. The `BLDCDriver` Trait

`BLDCDriver` extends `FOCDriver` with 3-phase voltage output modulation:

```rust
pub trait BLDCDriver: FOCDriver {
    /// Apply phase voltages (Va, Vb, Vc) in Volts
    fn set_pwm(&mut self, u_a: f32, u_b: f32, u_c: f32);

    /// Apply individual phase disable states (useful for 120° trapezoidal mode)
    fn set_phase_state(&mut self, sa: PhaseState, sb: PhaseState, sc: PhaseState);
}
```

---

## 3. The `StepperDriver` Trait

Controls 2-phase H-bridge inverters:

```rust
pub trait StepperDriver: FOCDriver {
    /// Apply Phase A and Phase B voltages (in Volts)
    fn set_pwm(&mut self, u_a: f32, u_b: f32);
}
```
