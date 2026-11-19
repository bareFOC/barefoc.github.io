---
layout: default
title: "Kinematics & Transmissions"
nav_order: 5
parent: "Actuators & Motors"
permalink: /actuators/kinematics/
---

# Kinematics & Transmissions (`barefoc::kinematics`)

The optional `kinematics` module (enabled via `features = ["kinematics"]`) provides coordinate transformations, gear transmission models, and joint calibration routines for robotic actuators and multi-axis mechanisms.

---

## 1. Transmission Models

Robotic actuators routinely pair motors with mechanical transmissions such as harmonic drives, planetary gearboxes, or lead screws.

### Gear Reductions & Transmissions:
The `Transmission` trait maps motor-side coordinates $(\theta_m, \omega_m, \tau_m)$ to output joint coordinates $(\theta_j, \omega_j, \tau_j)$:

$$\theta_j = \frac{\theta_m}{\text{ratio}}, \quad \omega_j = \frac{\omega_m}{\text{ratio}}, \quad \tau_j = \tau_m \cdot \text{ratio} \cdot \eta$$

Where $\eta$ is transmission mechanical efficiency.

```rust,no_run
use barefoc::kinematics::transmission::GearReduction;

// 100:1 planetary gearbox with 92% mechanical efficiency
let gearbox = GearReduction::new(100.0, 0.92);

let joint_angle = gearbox.motor_to_joint_angle(motor_shaft_angle);
let motor_torque = gearbox.joint_to_motor_torque(desired_joint_torque);
```

### Lead Screw Linear Actuators:
Translates rotary motion into linear displacement:

$$x = \frac{\theta_m \cdot \text{pitch}}{2\pi}$$

---

## 2. Multi-DOF Coordinate Transformations

BareFOC includes forward and inverse kinematics solvers for coupled joints:
- **Differential Drive / Differential Wrist**: Two motors drive two output axes through bevel gears ($J_1 = M_1 + M_2, J_2 = M_1 - M_2$).
- **Pan-Tilt Mechanism**: 2-DOF spherical coordinate mapping with singularity handling.

---

## 3. Joint Calibration & Backlash Compensation

Mechanical gearboxes possess backlash (hysteresis). `barefoc::kinematics::calibration` includes:
- **Hysteresis Deadband Compensation**: Dynamically injects an offset when reversing direction to eliminate positioning error.
- **Automated Mechanical Homing**: Low-speed current-limited stall detection against hard mechanical stops to zero joint references.
