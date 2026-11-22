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

5. [**AnySensor Dynamic Dispatch**](/sensors/any_sensor/):
   Zero-heap `AnySensor` enum dispatcher allowing runtime selection between I2C, SPI, QEI encoder, Hall, and virtual sensors driven by `ControlTable`'s `REG_MOTORx_SENSOR_TYPE`.

6. [**Multi-Zone Temperature Sensing**](/sensors/temperature/):
   `TemperatureSensor` trait, NTC thermistor and factory-calibrated MCU drivers, the MOSFET / MCU / motor register map, per-zone fault bits, and the `TM` / `TU` / `TW` / `LT` Commander commands.
