---
layout: default
title: "Low-Side Current Sensing"
nav_order: 3
parent: "Current Sensing"
permalink: /current_sensing/lowside/
---

# Low-Side Current Sensing

Low-side current sensing places shunt resistors between the source of each low-side MOSFET and power ground.

```
High-Side FET ────┐
                  ├───► Motor Phase
Low-Side FET  ────┤
                  │
             [ Shunt ]
                  │
                 GND
```

---

## Operating Principle & ADC Synchronization

Current flows through a low-side shunt **only when the corresponding low-side MOSFET is actively conducting**:
- When the high-side FET is ON, no current passes through the low-side shunt.
- When the low-side FET is ON, phase current returns to ground through the shunt.

Therefore, the ADC conversion trigger **must be strictly synchronized** to the timer counter midpoint when all low-side FETs are turned on simultaneously (center of the bottom PWM switching period).

```
PWM Carrier (Center-Aligned)
     ▲
     │    /\        /\        /\
     │   /  \      /  \      /  \
     │  /    \    /    \    /    \
     └─/──────\──/──────\──/──────\──► Time
               ▲
               │ ADC Trigger Point (Timer Underflow / Center)
```

Microcontroller hardware timer triggers (e.g., STM32 `TIM1_TRGO` firing ADC injected conversions) are supported natively in `barefoc-hal`.
