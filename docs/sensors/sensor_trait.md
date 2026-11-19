---
layout: default
title: "The Sensor Trait"
nav_order: 1
parent: "Sensors & Encoders"
permalink: /sensors/sensor_trait/
---

# The `Sensor` Trait

All position and velocity sensors in BareFOC implement the `Sensor` trait defined in `barefoc::traits::sensor::Sensor`.

---

## Trait Definition

```rust
pub trait Sensor {
    /// Read the raw mechanical shaft angle in radians within [0, 2π)
    fn get_mechanical_angle(&mut self) -> f32;

    /// Update internal tracking state (unwraps angle, updates velocity calculation)
    fn update(&mut self);

    /// Get continuous multi-turn shaft angle in radians (-∞ to +∞)
    fn get_angle(&mut self) -> f32;

    /// Get current shaft rotational velocity in radians/second
    fn get_velocity(&mut self) -> f32;

    /// Reset multi-turn counter to zero at current position
    fn reset_angle(&mut self);
}
```

---

## Multi-Turn Angle Unwrapping

Most absolute magnetic sensors report raw angles bounded strictly within $[0, 2\pi)$. For position servo control across multiple revolutions, BareFOC maintains a continuous multi-turn accumulator:

$$\Delta\theta = \theta_{\text{raw}}[k] - \theta_{\text{raw}}[k-1]$$

- If $\Delta\theta < -\pi$, a forward wrap occurred: `full_rotations += 1`.
- If $\Delta\theta > +\pi$, a reverse wrap occurred: `full_rotations -= 1`.

The continuous angle is then computed as:

$$\theta_{\text{continuous}} = \theta_{\text{raw}} + 2\pi \cdot \text{full\_rotations}$$

---

## Velocity Estimation

Shaft angular velocity is calculated on each `update()` call:

$$\omega = \frac{\Delta\theta}{\Delta t}$$

When sampling at high frequencies or using sensors with finite bit depth (e.g., 12-bit or 14-bit resolution), quantization steps cause discrete jumps in $\Delta\theta$. BareFOC conditions the velocity estimate using an internal first-order lowpass filter (`LPF_velocity`) to ensure smooth derivative action in velocity PID loops.
