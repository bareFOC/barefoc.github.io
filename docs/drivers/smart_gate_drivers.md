---
layout: default
title: "Smart Gate Drivers (drv83xx-rs)"
nav_order: 3
parent: "Drivers & Inverters"
permalink: /drivers/smart_gate_drivers/
---

# Smart Gate Drivers (`drv83xx-rs`)

Modern smart gate drivers integrate serial communication buses (SPI/tSPI) to configure current sense amplifier (CSA) gains, set dead-time values, detect over-temperature/over-current faults, and manage charge pumps.

The `drv83xx-rs` crate within the BareFOC workspace provides dedicated drivers for Texas Instruments motor driver ICs.

---

## 1. Texas Instruments DRV8316

The DRV8316 integrates 40V, 8A peak low-$R_{DS(on)}$ MOSFETs and three current sense amplifiers:
- 16-bit SPI register configuration.
- Selectable CSA gain (150 mV/A, 300 mV/A, 600 mV/A, 1200 mV/A).
- Active dynamic braking and hardware cycle-by-cycle overcurrent protection (OCP).

```rust,no_run
use drv83xx_rs::drv8316::{DRV8316, CsaGain, PwmMode};

let mut drv = DRV8316::new(spi_device, nsleep_pin);
drv.init();

// Configure 3x PWM hardware mode with 300 mV/A CSA gain
drv.set_pwm_mode(PwmMode::Mode3xPwm);
drv.set_csa_gain(CsaGain::Gain300mV_A);
drv.enable();
```

---

## 2. Texas Instruments DRV8311

The DRV8311 is a compact, ultra-low power 3-phase gate driver supporting advanced SPI protocols:
- **24-bit SPI Mode**: Standard command-address-data framing for MCU register configuration.
- **32-bit tSPI Mode**: Ultra-fast single-cycle read/write framing for real-time telemetry streaming.
- **Substrate Cross-Talk Decoupling**: High-frequency noise suppression between adjacent half-bridges.
- **Hardware PWMG**: Automated PWM generation and fault latching.

```rust,no_run
use drv83xx_rs::drv8311::{DRV8311, SpiMode};

let mut drv = DRV8311::new(spi_device, nsleep_pin, SpiMode::Mode24Bit);
drv.wake();
drv.clear_faults();
```
