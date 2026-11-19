---
layout: default
title: "Modulation Schemes"
nav_order: 3
parent: "Theory & Mathematics"
permalink: /theory/modulation/
---

# Modulation Schemes

Once the desired stator voltage vector $(V_\alpha, V_\beta)$ is calculated, the power inverter must convert these continuous references into pulse-width modulated (PWM) switching signals. BareFOC supports three primary modulation strategies:

1. **Space Vector PWM (SVPWM)** (Recommended)
2. **Sine PWM (`SinePWM`)**
3. **Trapezoidal Modulation (120° / 150°)**

---

## 1. Space Vector PWM (SVPWM)

Space Vector PWM synthesizes the target voltage vector by selecting adjacent inverter switching states and modulating their dwell times within a switching period $T_{pwm}$.

```
                 V2 (010)        V1 (110)
                     \          /
                      \ Sector /
                       \  II  /  Sector I
                        \    /
            V3 (011) -----\--/------ V0, V7 (Null) ---- V6 (100)
                        /    \
                       / Sector\ Sector VI
                      /  IV   \
                     /          \
                 V4 (001)        V5 (101)
```

### Key Advantages of SVPWM:
- **DC Bus Utilization**: Standard sinusoidal PWM saturates at $V_{\text{max}} = \frac{V_{dc}}{2} \approx 0.50 V_{dc}$. SVPWM utilizes up to $V_{\text{max}} = \frac{V_{dc}}{\sqrt{3}} \approx 0.577 V_{dc}$ — a **15.4% increase in achievable motor speed and torque** for the same supply voltage.
- **Harmonic Distortion**: Center-aligned switching cancels third-order harmonics, reducing acoustic noise and coil heating.

### Rust Implementation:
```rust
use barefoc::math::svpwm;

// Computes duty cycles (Ta, Tb, Tc) normalized to [0.0, 1.0]
let (duty_a, duty_b, duty_c) = svpwm(v_alpha, v_beta, v_bus);
```

---

## 2. Sine PWM (`SinePWM`)

Sine PWM calculates sinusoidal phase voltages directly using the Inverse Clarke transform:

$$\begin{aligned}
V_a &= V_q \sin(\theta_e) + \frac{V_{bus}}{2} \\
V_b &= V_q \sin\left(\theta_e - \frac{2\pi}{3}\right) + \frac{V_{bus}}{2} \\
V_c &= V_q \sin\left(\theta_e + \frac{2\pi}{3}\right) + \frac{V_{bus}}{2}
\end{aligned}$$

Sine PWM requires fewer trigonometric calculations than SVPWM and produces pure sinusoidal phase-to-neutral voltages, but it cannot access the top 15.4% of the DC bus voltage.

---

## 3. Trapezoidal Modulation

Trapezoidal commutation emulates conventional 6-step square-wave commutation:
- **120° Mode**: Only two phases conduct current at any moment; the third phase is left floating (high-impedance / Hi-Z).
- **150° Mode**: Phases conduct for 150 electrical degrees with smooth transitions.

Trapezoidal modulation is primarily used for sensorless back-EMF zero-crossing detection or low-resource microcontrollers that cannot compute high-frequency trigonometric operations.
