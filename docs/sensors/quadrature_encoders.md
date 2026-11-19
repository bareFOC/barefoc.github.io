---
layout: default
title: "Quadrature Encoders"
nav_order: 3
parent: "Sensors & Encoders"
permalink: /sensors/quadrature_encoders/
---

# Quadrature Encoders

Quadrature encoders generate two square-wave channels ($A$ and $B$) phased 90° apart, plus an optional index channel ($Z$) that pulses once per physical revolution.

---

## Quadrature Decoding Principles

By monitoring state transitions on both channels ($4\times$ decoding), every full optical line or magnetic pole produces 4 count edges:

$$\text{CPR (Counts Per Revolution)} = 4 \times \text{PPR (Pulses Per Revolution)}$$

```
Channel A:  ───┐   ┌───┐   ┌───┐   ┌───
               └───┘   └───┘   └───┘
Channel B:  ─────┐   ┌───┐   ┌───┐   ┌─
                 └───┘   └───┘   └───┘
Count:      0  1 2 3 4 5 6 7 8 ...
```

---

## 1. Microcontroller Hardware Timer Encoders (`barefoc-hal`)

Directly counting quadrature pulses via software interrupts degrades at high speeds (e.g. 10,000 RPM $\times$ 4096 CPR = 682,000 interrupts/sec).

BareFOC offloads decoding to microcontroller hardware timer quadrature decoder peripherals:

### STM32 Advanced & General-Purpose Timers (`barefoc-stm32`):
```rust,no_run
use barefoc_stm32::encoder::Stm32Encoder;
use barefoc::traits::sensor::Sensor;

// Connect to hardware timer (e.g. TIM2, TIM3, TIM4) configured in encoder mode
let mut encoder = Stm32Encoder::new(timer_channel, 1024 /* PPR */);

encoder.update();
let angle = encoder.get_angle();
```

### ESP32 Pulse Counter (`barefoc-esp32`):
Uses the dedicated ESP32 PCNT (Pulse Counter) hardware unit:
```rust,no_run
use barefoc_esp32::pcnt::Esp32PcntSensor;

let mut encoder = Esp32PcntSensor::new(pcnt_unit, pin_a, pin_b, 1024);
```

---

## 2. Software Encoder Fallback (`barefoc::sensors::Encoder`)

For microcontrollers lacking free hardware encoder timer channels, BareFOC includes `barefoc::sensors::Encoder` which updates position counters via external GPIO edge interrupts (`EXTI`).
