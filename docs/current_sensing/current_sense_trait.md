---
layout: default
title: "The CurrentSense Trait"
nav_order: 1
parent: "Current Sensing"
permalink: /current_sensing/current_sense_trait/
---

# The `CurrentSense` Trait

All current sensing implementations in BareFOC implement the `CurrentSense` trait defined in `barefoc::traits::current_sense::CurrentSense`.

---

## Trait Definition

```rust
pub trait CurrentSense {
    /// Initialize hardware ADCs, zero offsets, and verify gain signs
    fn init(&mut self) -> Result<(), ()>;

    /// Read instantaneous phase currents (Ia, Ib, Ic) in Amperes
    fn get_phase_currents(&mut self) -> (f32, f32, f32);

    /// Compute orthogonal stationary currents (I_alpha, I_beta) via Clarke transform
    fn get_fo_currents(&mut self) -> (f32, f32);

    /// Calculate instantaneous motor electrical torque in Newton-meters
    fn get_torque(&mut self, kv_rating: f32) -> f32;
}
```

---

## Calibration Routine

During motor startup (`motor.init_foc()`), BareFOC performs an automated current sense alignment:

1. **Zero-Offset Measurement**: With the inverter bridge disabled ($I = 0$), the ADC averages thousands of samples to record the precise DC quiescent voltage of each amplifier channel ($V_{\text{offset}}$).
2. **Gain Sign Alignment**: Applies a known positive voltage vector along Phase A and measures current. If the measured current is negative, the driver automatically flips the phase gain sign, eliminating manual wiring adjustments.
