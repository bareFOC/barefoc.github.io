---
layout: default
title: "Inline Current Sensing"
nav_order: 2
parent: "Current Sensing"
permalink: /current_sensing/inline/
---

# Inline Current Sensing

Inline current sensing places precision low-resistance shunt resistors (e.g. $10\text{ m}\Omega$ to $50\text{ m}\Omega$) directly in series with the motor phase leads.

```
Inverter Phase A ────[ Shunt Resistor ]────► Motor Winding A
                              │
                    High Common-Mode CSA
                              │
                             ADC
```

---

## Operating Characteristics

- **High Common-Mode Rejection Ratio (CMRR) Required**: Because the shunt switches between $0\text{ V}$ and $V_{\text{bus}}$ at high PWM frequencies (up to 40 kHz with $\frac{dV}{dt} \ge 5\text{ V/ns}$), the current sense amplifier (e.g. INA240, AD8418) must possess extremely high AC CMRR ($>90\text{ dB}$).
- **Continuous Sampling**: Unlike low-side sensing, the phase current flows through the shunt throughout the entire electrical cycle. The ADC can sample at any arbitrary moment without needing synchronization to timer reload events.

---

## Rust Usage:
```rust,no_run
use barefoc::current_sense::inline::InlineCurrentSense;

// Shunt: 10 mOhm, CSA Gain: 50 V/V -> Total gain = 50 / 0.010 = 5000 V/A
let mut cs = InlineCurrentSense::new(0.010, 50.0, adc_pin_a, adc_pin_b, None);
cs.init();
```
