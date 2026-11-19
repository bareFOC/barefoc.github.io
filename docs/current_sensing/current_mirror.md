---
layout: default
title: "Current Mirror Sensing"
nav_order: 4
parent: "Current Sensing"
permalink: /current_sensing/current_mirror/
---

# Current Mirror Sensing

Smart gate drivers and modern monolithic H-bridge ICs often include integrated current mirrors directly on the silicon die (such as Texas Instruments SENSEFET technology).

---

## Operating Characteristics

- **Zero Resistor Heat Dissipation**: Replaces bulky, heat-dissipating milliohm shunt resistors with a scaled ratio transistor (e.g. 1:10,000 ratio).
- **Proportional Voltage Output**: Generates an analog voltage directly proportional to instantaneous phase current.
- **`CurrentMirrorSense`**: BareFOC provides the `CurrentMirrorSense` struct in `barefoc-drivers` to normalize current mirror voltage curves directly into the standard `CurrentSense` trait.

```rust,no_run
use barefoc_drivers::dc::current_mirror::CurrentMirrorSense;

// Scale: 1200 mV per Ampere, zero-current quiescent at 1.65 V
let mut mirror_sense = CurrentMirrorSense::new(adc_channel, 1.2, 1.65);
mirror_sense.init();
```
