---
layout: default
title: "Magnetic Encoders"
nav_order: 2
parent: "Sensors & Encoders"
permalink: /sensors/magnetic_sensors/
---

# Magnetic Encoders

Magnetic angle sensors measure the orientation of a diametrically magnetized disc attached to the motor shaft. They provide absolute position across power cycles and withstand vibration and dust.

---

## Supported Magnetic Sensor Interfaces

BareFOC and `barefoc-drivers` provide ready-to-use drivers across all common digital interfaces:

| Sensor IC | Resolution | Interface | Driver Crate / Struct | Key Features |
|---|---|---|---|---|
| **AS5600** | 12-bit (4096 CPR) | I2C | `barefoc_drivers::encoders::AS5600` | Low cost, AGC magnet strength status |
| **AS5048A** | 14-bit (16384 CPR) | SPI | `barefoc_drivers::encoders::AS5048A` | Odd parity check, error flags |
| **AS5048B** | 14-bit (16384 CPR) | I2C | `barefoc_drivers::encoders::AS5048B` | Configurable I2C address pins |
| **AS5047P / AS5047D** | 14-bit (16384 CPR) | SPI | `barefoc_drivers::encoders::AS5047P` | DAEC dynamic angle error compensation up to 28,000 RPM |
| **MT6701** | 14-bit (16384 CPR) | SSI / SPI | `mt6xxx_rs::MT6701` | High-speed simplex SSI, hardware CRC-6 checking |
| **Generic SSI** | 10–18 bit | SSI | `barefoc_drivers::encoders::GenericSSI` | Configurable bit depth and framing |
| **Analog / PWM** | Continuous | ADC / Timer | `barefoc::sensors::MagneticSensorAnalog` | Linear voltage to angle mapping |

---

## 1. MagnTek MT6701 via SSI (`mt6xxx-rs`)

The MT6701 is an optimal sensor for high-performance BLDC servos due to its low latency and robust simplex Synchronous Serial Interface (SSI):

```rust,no_run
use mt6xxx_rs::{MT6701, SsiConfig};
use barefoc::traits::sensor::Sensor;

// Initialize SPI peripheral in mode 1 or mode 2
let mut encoder = MT6701::new_ssi(spi_device, SsiConfig::default());

// Reads 14-bit angle, verifies CRC-6 checksum, updates multi-turn tracker
encoder.update();
let angle_rad = encoder.get_angle();
let vel_rad_s = encoder.get_velocity();
```

---

## 2. ams AS5047P via SPI (`barefoc-drivers`)

The AS5047P includes DAEC (Dynamic Angle Error Compensation) which internally cancels propagation delay at high rotational velocities:

```rust,no_run
use barefoc_drivers::encoders::AS5047P;
use barefoc::traits::sensor::Sensor;

let mut sensor = AS5047P::new(spi_device);
sensor.init();

sensor.update();
let shaft_angle = sensor.get_mechanical_angle();
```

---

## 3. ams AS5600 via I2C (`barefoc-drivers`)

```rust,no_run
use barefoc_drivers::encoders::AS5600;

let mut sensor = AS5600::new(i2c_bus);
let magnet_detected = sensor.is_magnet_detected();
let agc_gain = sensor.get_agc(); // Gain verification for magnet air gap
```
