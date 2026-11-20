---
layout: default
title: "Sensors & Encoders"
nav_order: 6
has_children: true
permalink: /sensors/
---

# Sensors & Encoders

Field Oriented Control requires continuous knowledge of rotor position to maintain the stator magnetic flux vector perpendicular to the rotor magnets. BareFOC abstracts all position and velocity sensing behind the `Sensor` trait.

---

## Sensor Topics

1. [**The `Sensor` Trait**](/sensors/sensor_trait/):
   The zero-cost trait contract: `get_mechanical_angle()`, `get_angle()`, `get_velocity()`, `update()`, angle unwrapping, and velocity filtering.

2. [**Magnetic Encoders**](/sensors/magnetic_sensors/):
   Absolute angle sensor ICs over I2C (AS5600), SPI (AS5048A, AS5047P), SSI (`MT6701` in `mt6xxx-rs`), and analog/PWM interfaces.

3. [**Quadrature Encoders**](/sensors/quadrature_encoders/):
   Incremental optical and magnetic encoders with A/B/Z index channels. Explains software decoding and microcontroller hardware timer decoders (`Stm32Encoder`, `Esp32PcntSensor`).

4. [**Hall Effect Sensors**](/sensors/hall_sensors/):
   Digital 3-Hall commutation sensors and modern 3D Hall-effect flux sensors (`tmag-rs` with the TI TMAG5273).
