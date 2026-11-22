---
layout: default
title: "AnySensor Dynamic Dispatch"
nav_order: 5
parent: "Sensors & Encoders"
permalink: /sensors/any_sensor/
---

# AnySensor Dynamic Multi-Interface Dispatcher

In BareFOC's "Flash Once and Forget" architecture, embedded actuator boards are flashed at manufacturing time. Actuator hardware often supports multiple sensor interfaces:
- Onboard magnetic sensors (e.g. AS5600 over I2C or MT6701 over SPI).
- Expansion connectors for external optical or magnetic quadrature encoders (timer QEI).
- 3-phase digital Hall effect position sensors.
- Open-loop testing modes with virtual angles.

Because standard SimpleFOC generic monomorphization binds `BLDCMotor<'clk, S, ...>` to a single concrete sensor type `S` at compile time, users traditionally had to recompile and reflash firmware to change sensor hardware.

The `barefoc-stm32` HAL provides [`AnySensor`], a zero-heap, `#![no_std]` dynamic enum dispatcher that wraps I2C, SPI, timer QEI, Hall, and virtual sensors. It implements [`Sensor<C>`] with fully inlined method delegation, allowing runtime interface selection via `ControlTable`'s `REG_MOTORx_SENSOR_TYPE` register.

---

## 1. AnySensor Architecture

`AnySensor` is defined as a fixed-size enum generic over the underlying bus and timer peripheral types:

```rust
pub enum AnySensor<
    I2C: embedded_hal::i2c::I2c,
    SPI: embedded_hal::spi::SpiDevice<u16>,
    QEI: GeneralInstance4Channel,
    C: Clock = DwtClock,
> {
    /// I2C magnetic position sensor (AS5600, AS5048B, TMAG5273)
    I2c(MagneticSensorI2C<C, I2C>),
    /// SPI magnetic position sensor (MT6701, AS5047P, MA730)
    Spi(MagneticSensorSPI<C, SPI>),
    /// Hardware timer quadrature encoder (Stm32Encoder)
    Encoder(Stm32Encoder<'static, QEI>),
    /// 3-phase digital Hall effect sensor
    Hall(HallSensor<C>),
    /// Virtual dummy sensor for open-loop diagnostics
    Virtual(DummySensor),
    /// TMAG5273 3D magnetic angle sensor via I2C
    #[cfg(feature = "tmag")]
    Tmag5273(tmag_rs::barefoc::Tmag5273Sensor<I2C>),
    /// Safe unconfigured fallback returning 0.0 rad
    Unconfigured(SensorBase),
}
```

### Zero-Allocation Guarantees
- **No Dynamic Memory**: The enum is sized statically on the stack or in static memory (`max(size_of::<Variant>()) + discriminant` $\approx 64$ bytes).
- **Inlined Delegation**: All trait methods (`get_sensor_angle`, `base`, `base_mut`, `init`, `update`, `get_mechanical_angle`, `get_angle`, `get_velocity`, `needs_search`) use direct compiler inlining (`#[inline]`), eliminating vtable indirection in the high-speed FOC commutation loop.
- **In-Place Mutation**: Switching between sensor variants occurs in place via `sensor.switch_variant(...)` or `sensor.switch_to_virtual(...)` without memory reallocation.

---

## 2. Sensor Type Selection Matrix

The `ControlTable` register `REG_MOTORx_SENSOR_TYPE` (`0x13` relative to motor base) maps numerical IDs (`0..7`) to sensor configurations:

| Type ID | Enum Variant | Target Hardware & Protocol | Default Address / Settings |
|:---:|---|---|---|
| `0` | `DefaultOnboard` | Board BSP default interface | Auto-detected from available bus |
| `1` | `As5600` | AMS AS5600 12-bit magnetic sensor | I2C address `0x36`, angle reg `0x0C` |
| `2` | `As5048b` | AMS AS5048B 14-bit magnetic sensor | I2C address `0x40`, angle reg `0xFE` |
| `3` | `Mt6701` | MagnTek MT6701 14-bit SSI/SPI sensor | SPI Mode 0, 1 MHz, angle reg `0x03` |
| `4` | `As5047p` | AMS AS5047P / MA730 14-bit SPI sensor | SPI Mode 1, 1 MHz, angle reg `0x3FFF` |
| `5` | `EncoderQei` | Hardware Timer Quadrature Encoder | Hardware 4x decoding, timer peripheral QEI |
| `6` | `Hall` | 3-phase digital Hall effect sensors | 3 digital GPIO inputs with EXTI interrupts |
| `7` | `Virtual` | `DummySensor` open-loop angle source | User-settable virtual angle |
| `8` | `Tmag5273` | TI TMAG5273 16-bit 3D magnetic sensor | I2C candidate addresses (`0x35`, `0x22`, `0x78`, `0x44`) |

---

## 3. Usage & Runtime Switching Example

### Initializing and Building from Factory Type

```rust,no_run
use barefoc_stm32::{AnySensor, SensorType, Stm32Encoder};
use barefoc::traits::sensor::Sensor;

// Initialize board peripherals
let i2c = // board I2C peripheral ...
let spi = // board SPI peripheral ...

// Instantiate AnySensor linked to ControlTable REG_MOTORx_SENSOR_TYPE
let sensor_type_id = control_table.motor_sensor_type(0); // e.g. 1 for AS5600
let mut any_sensor = AnySensor::build_from_type(
    sensor_type_id,
    Some(i2c),
    Some(spi),
    None, // encoder
    None, // hall
).expect("Failed to initialize requested sensor variant");

// Pass any_sensor directly into motor FOC loop
motor.set_sensor(&mut any_sensor);
```

### In-Place Runtime Variant Switching

Actuator diagnostics and calibration routines can switch between sensors on the fly:

```rust,no_run
// Switch to virtual sensor for open-loop testing
any_sensor.switch_to_virtual(0.0);
assert_eq!(any_sensor.sensor_type(), Some(SensorType::Virtual));

// Update virtual angle
if let Some(virtual_sensor) = any_sensor.as_virtual_mut() {
    virtual_sensor.set_angle(core::f32::consts::PI);
}
assert_eq!(any_sensor.get_sensor_angle(), core::f32::consts::PI);

// Switch to hardware quadrature encoder
let encoder = Stm32Encoder::new(tim3, ch1_pin, ch2_pin, 1000);
any_sensor.switch_variant(AnySensor::from_encoder(encoder));
assert_eq!(any_sensor.sensor_type(), Some(SensorType::EncoderQei));
```
