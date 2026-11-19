---
layout: default
title: "Theory & Mathematics"
nav_order: 3
has_children: true
permalink: /theory/
---

# Theory & Mathematics of Field Oriented Control

Field Oriented Control (FOC) is a vector control methodology for brushless motors that decouples electromagnetic torque production from magnetic rotor flux. By projecting three-phase alternating quantities into an orthogonal two-coordinate frame rotating synchronously with the rotor, AC machines are controlled with the linear simplicity of a separately excited DC machine.

---

## Core Theoretical Concepts

The BareFOC mathematical engine is divided into four primary topics:

1. [**FOC Principles**](/theory/foc_principles/):
   The electromagnetic relationship between stator magnetic flux and permanent magnet rotor poles. Explains direct axis ($d$) flux, quadrature axis ($q$) torque, and electrical angle calculation ($\theta_e = PP \cdot \theta_m$).

2. [**Coordinate Transforms**](/theory/transforms/):
   The Clarke transform ($a, b, c \to \alpha, \beta$) and Park transform ($\alpha, \beta \to d, q$), along with their inverse operations and normalized angle mathematics using `libm`.

3. [**Modulation Schemes**](/theory/modulation/):
   Voltage synthesis strategies including Space Vector PWM (SVPWM), standard Sine PWM (`SinePWM`), and 120° / 150° Trapezoidal commutation.

4. [**Controllers & Filters**](/theory/control_loops/):
   Cascaded control topologies (Torque $\to$ Velocity $\to$ Angle), the `PIDController` algorithm with anti-windup clamping and derivative filtering, and the first-order `LowPassFilter`.
