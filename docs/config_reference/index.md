---
layout: default
title: "Configuration Reference"
nav_order: 4
has_children: false
permalink: /config_reference/
---

# Configuration Reference

This document is a reference for all TOML configuration sections used in BareFOC:

1. **Board Hardware Configuration (`barefoc.toml`)**: Declarative PCB definition read at compile time by `build.rs` and `bareConfig` (`bareconfig::bsp::schema`).
2. **Runtime Commissioning Configuration (`servo.toml`)**: Declarative joint/kinematics profile applied over MIPI I3C via `configure_fishyservo_i3c.py --config <file.toml> --save` and persisted in flash (Bank 2 Sectors 14 & 15).

> **Note:** Key names and types below match `bareConfig/src/bsp/schema.rs` and `configure_fishyservo_i3c.py`. Sections marked **optional** use safe defaults or are auto-derived if omitted.

---

## Board Hardware Sections (`barefoc.toml`)

### `[board]`

Defines the board identity and silicon target.

```toml
[board]
name = "fishyservo-stm32h5"
#   Human-readable board identifier. Required.
description = "FishyServo Quad H-Bridge Multi-Topology Servo (STM32H503KB)"
#   Short description shown in bareConfig and build metadata. Required.
mcu = "stm32h503kbu6"
#   Target MCU part number (used to auto-derive [memory] layout). Required.
family = "stm32h5"
#   HAL family identifier (e.g., "stm32h5", "stm32g4", "stm32f4", "esp32", "ch32"). Required.
system_clock_hz = 250000000
#   Core system clock frequency in Hz. Required.
```

---

### `[memory]` *(optional — auto-derived)*

Flash, RAM, bootloader, application slot, and NVM persistence layout.
When `[memory]` (or any individual field inside it) is omitted, `MemoryLayout::derive()` automatically computes the full layout from `board.mcu` and `[bootloader]`:

| MCU Prefix | Flash Origin / Size | RAM Origin / Size | `bareboot` App Origin / Size | `bareboot` NVM Origin / Size |
| :--- | :--- | :--- | :--- | :--- |
| `stm32h503*` | `0x08000000` / `128 KiB` | `0x20000000` / `32 KiB` | `0x08004000` / `88 KiB` | `0x0801C000` / `16 KiB` |
| `stm32g431*` | `0x08000000` / `128 KiB` | `0x20000000` / `32 KiB` | `0x08004000` / `88 KiB` | `0x0801C000` / `16 KiB` |
| `stm32g474*` | `0x08000000` / `512 KiB` | `0x20000000` / `128 KiB` | `0x08004000` / `472 KiB` | `0x0807C000` / `16 KiB` |
| `stm32f405*` / `stm32f407*` | `0x08000000` / `1024 KiB` | `0x20000000` / `128 KiB` | `0x08004000` / `984 KiB` | `0x080FC000` / `16 KiB` |
| `stm32f446*` | `0x08000000` / `512 KiB` | `0x20000000` / `128 KiB` | `0x08004000` / `472 KiB` | `0x0807C000` / `16 KiB` |

*(For `bootloader.type = "bareboot"`, 8 KiB before `nvm_origin` is automatically reserved for the BareBoot CommitStore.)*

```toml
[memory]
#flash_origin = 0x08000000
#flash_size = 131072
#ram_origin = 0x20000000
#ram_size = 32768
#bootloader_origin = 0x08000000
#bootloader_size = 16384
#app_origin = 0x08004000
#app_size = 90112
#nvm_origin = 0x0801C000
#nvm_size = 16384
```

---

### `[communication.<name>]` *(optional)*

Semantic named communication ports (`[communication.host]`, `[communication.sensor_bus]`, `[communication.encoder_bus]`, etc.) with explicit `role` (`"target"`, `"master"`, `"both"`), `bootloader` flag, optional `peripheral` disambiguator, and optional I2C `mux` / `mux_address`. Pin assignments support Klipper-style `!`/`^`/`~` modifiers, and schema validation rejects any GPIO pin conflicts across active ports. Shared identically with `bareboot.toml`.

```toml
[communication.host]
protocol = "i3c"
#   Transport protocol: "i3c", "i2c", "spi", "can", "uart", "usb", or "rtt".
role = "target"
#   Port role: "target" (host-facing), "master" (downstream sensors/drivers), or "both".
bootloader = true
#   Allow bootloader access on this port (valid only when role is "target" or "both").
peripheral = "I3C1"
#   Optional MCU peripheral disambiguator ("I3C1", "FDCAN1", "USART1", "I2C1", "SPI1").
tx_scl_pin = "PB3"
#   TX / SCL / CAN_TX pin.
rx_sda_pin = "PB4"
#   RX / SDA / CAN_RX pin.
#can_enable_pin = "PC13"
#   Optional CAN transceiver enable pin.
bitrate = 1000000
#   Required for async "uart" / "can"; optional host hint for "i3c" / "i2c" / "spi".
address = 0x40
#   Static/proposed 7-bit I2C or I3C target address. Optional.

[communication.sensor_bus]
protocol = "i2c"
role = "master"
peripheral = "I2C1"
tx_scl_pin = "PB6"
rx_sda_pin = "PB7"
```

---

### `[bootloader]` *(optional)*

Configures bootloader handoff, boot window, safe startup pins, and SRAM mailbox synchronization. Shared identically with `bareboot.toml`.

```toml
[bootloader]
enabled = true
#   Whether the application links behind a bootloader slot. Default: true.
type = "bareboot"
#   Bootloader type: "bareboot" or "katapult". Default: "bareboot".
boot_window_ms = 250
#   Bootloader wait window before jumping to app, in milliseconds. Default: 250.
preserve_nvm = true
#   Whether flashing preserves the application NVM sectors. Default: true.
#initial_safe_pins = ["!PA9", "^PB12"]
#   Optional Klipper-style safe startup pins (auto-derived from [gate_driver] if omitted).
#status_led = "!PA1"
#   Optional Klipper-style single status LED pin ("!<PIN>" for active-low, "<PIN>" for active-high).
mailbox_address = 0x20007FE0
#   SRAM address of the soft-reset / boot-loop mailbox. Default: 0x20007FF8.
mailbox_magic = 0x5984E3FA6CA1589B
#   64-bit magic signature requesting bootloader entry. Default: 0x5984E3FA6CA1589B.
```

#### Status LED Blink Patterns

When `[bootloader].status_led` is configured, BareBoot drives deterministic single-LED patterns in priority order:

| Priority | State | Condition | Pattern (`ms % 1000`) |
| --- | --- | --- | --- |
| 1 | `Flashing` | Active programming (`RX_BLOCK` / `RX_EOF` / `COMPLETE`) | 10 Hz fast blink (50 ms on / 50 ms off) |
| 2 | `BootLoopError` | Boot-loop trap (`>= 3` fault resets) or flash error | Triple-blink (`0..80`, `160..240`, `320..400` ms) |
| 3 | `BlankApp` | No valid application vector or CRC32 in flash | Double-blink (`0..100`, `200..300` ms) |
| 4 | `BootWindow` | Waiting for host activity before auto-booting app | Solid ON |
| 5 | `Parked` | Valid app present, parked in bootloader mode | 1 Hz heartbeat (500 ms on / 500 ms off) |

---

### `[safety]`

Hard PCB protection ceilings and fault trip thresholds enforced by the FOC safety supervisor.

```toml
[safety]
max_voltage_v = 6.0
#   Hard maximum bus/phase voltage ceiling in Volts. Required.
#min_bus_voltage_v = 4.0
#   Optional under-voltage lockout threshold in Volts.
max_current_a = 2.0
#   Hard maximum phase/bus current ceiling in Amperes. Required.
#continuous_current_a = 1.5
#   Optional continuous RMS current rating in Amperes (<= max_current_a).
#i2t_limit = 10.0
#   Optional I2t thermal integration limit.
max_velocity_rad_s = 50.0
#   Maximum allowed angular velocity in rad/s. Required.
temp_limit_mosfet_c = 85.0
#   MOSFET over-temperature shutdown threshold in degrees Celsius. Optional.
#temp_limit_mcu_c = 105.0
#   Optional MCU die temperature shutdown threshold in degrees Celsius.
#temp_limit_motor_c = 100.0
#   Optional motor winding temperature shutdown threshold in degrees Celsius.
#stall_timeout_ms = 500
#comm_timeout_ms = 250
#runaway_timeout_ms = 200
#max_angle_jump_rad = 1.0
#max_enable_step_rad = 0.5
```

---

### `[defaults]` *(optional — factory fallback)*

Initial runtime operating limits (`voltage_limit <= safety.max_voltage_v`, `current_limit <= safety.max_current_a`, `velocity_limit <= safety.max_velocity_rad_s`), PID gains, and filter constants used when NVM is blank or invalid.

```toml
[defaults]
node_id = 64
#   Default 7-bit bus node ID. Default: 1.
pole_pairs = 1
#   Motor electrical pole pairs. Default: 7.
voltage_limit = 5.0
#   Initial runtime voltage limit in Volts (<= safety.max_voltage_v).
current_limit = 1.0
#   Initial runtime current limit in Amperes (<= safety.max_current_a).
velocity_limit = 50.0
#   Initial runtime velocity limit in rad/s (<= safety.max_velocity_rad_s).
pid_velocity_p = 0.5
#   Velocity PID proportional gain. Default: 0.25.
pid_velocity_i = 1.0
#   Velocity PID integral gain. Default: 3.5.
pid_velocity_d = 0.0
#   Velocity PID derivative gain. Default: 0.001.
pid_velocity_output_ramp = 100.0
#   Velocity PID output ramp rate (V/s or A/s). Default: 100.0.
pid_angle_p = 20.0
#   Position/angle PID proportional gain. Default: 25.0.
pid_angle_i = 0.0
#   Position/angle PID integral gain. Default: 0.0.
pid_angle_d = 0.1
#   Position/angle PID derivative gain. Default: 0.05.
lpf_velocity_tf = 0.02
#   Velocity low-pass filter time constant in seconds. Default: 0.015.
lpf_angle_tf = 0.0
#   Angle low-pass filter time constant in seconds. Default: 0.0.
```

---

### `[gate_driver]` and `[[gate_driver.channels]]`

Defines the power stage topology, PWM frequency, and per-channel bridge pins. All pin fields support Klipper-style modifiers (`!` inverted/active-low, `^` pull-up, `~` pull-down, e.g. `fault_pin = "^!PB12"`, `cs_pin = "!PC4"`).

When `[[gate_driver.channels]]` is populated (e.g. 4-channel H-bridge boards), `inh_a`, `inh_b`, and `inh_c` are optional and do not need dummy placeholders.

```toml
[gate_driver]
driver_type = "drv8411a"
#   Gate driver IC ("drv8411a", "drv8311", "drv8316", "3pwm", "6pwm", etc.). Required.
pwm_frequency_hz = 25000
#   PWM carrier frequency in Hz. Required.
pwm_timer = "TIM1+TIM3"
#   MCU timer peripheral(s) driving PWM outputs. Required.
#inh_a = "PA10"
#inh_b = "PA9"
#inh_c = "PA8"
#   Primary phase / high-side PWM pins (required when [[gate_driver.channels]] is empty).
#inl_a = "PB15"
#inl_b = "PB14"
#inl_c = "PB13"
#   Complementary / low-side or enable pins. Optional.
sleep_pin = "PA9"
#   Sleep / driver enable GPIO pin. Optional.
#fault_pin = "^!PB12"
#   Hardware nFAULT input pin (supports ^ pull-up and ! inverted). Optional.

[[gate_driver.channels]]
#   Optional per-channel H-bridge / half-bridge descriptors for multi-axis boards.
channel = 0
bridge = "D"
in1_pin = "PA9"
in2_pin = "PB0"
adc_pin = "PA5"
timer = "TIM1_CH2+TIM3_CH3"
```

---

### `[sensor.<name>]`

Named sensor instances (`[sensor.motor]`, `[sensor.output]`, `[sensor.s0]`, `[sensor.ext0]`, etc.) referencing a `[communication.<name>]` bus with `role = "master"` (or inline peripheral/pins). Shared SPI buses require distinct `cs_pin` entries; shared I2C buses can specify either distinct `(address, mux_channel)` pairs or a `addresses = [...]` candidate list for runtime factory-variant probing (e.g., FishyServo's 3 physical I2C1 connectors with TMAG5273 variants `[0x35, 0x22, 0x78, 0x44]`) with default `motor = 0..3`; external runtime-selectable connectors declare `supported = [...]` + `default` to emit `SUPPORTED_SENSORS_MASK` and expose `REG_SUPPORTED_SENSORS` over MultiBus.

```toml
[sensor.s0]
type = "tmag5273"
#   Fixed sensor type ("tmag5273", "mt6701", "mt6701_ssi", "as5047p", "as5600", "qei", "hall", "virtual").
bus = "sensor_bus"
#   References [communication.sensor_bus] (role = "master") or inline peripheral ("SPI2", "I2C1").
addresses = [0x35, 0x22, 0x78, 0x44]
#   Candidate 7-bit I2C addresses for runtime probing on a shared bus (or `address = 0x35` for a fixed address).
motor = 0
#   Default motor channel assignment (overridable at runtime via `REG_SENSOR_MAP` 0x238).
#cpr = 65536
#   Counts per revolution (auto-defaulted from sensor type when omitted).

# Example runtime-selectable external sensor connector:
#[sensor.ext0]
#default = "mt6701"
#supported = ["mt6701", "as5047p", "virtual"]
#bus = "encoder_bus"
#cs_pin = "!PA4"
```

---

### `[assets]` *(optional)*

Artifact filenames bundled for flashing tools and `bareConfig`.

```toml
[assets]
bootloader_bin = "bareboot-stm32h503-i3c-i2c.bin"
firmware_bin = "fishyservo_stm32h5_i3c.bin"
```

---

## Runtime Commissioning Sections (`configure_fishyservo_i3c.py`)

A flashed board is configured over MIPI I3C without recompiling using `configure_fishyservo_i3c.py --config <file.toml> --save`. You can pass either a standalone commissioning TOML or a board's `barefoc.toml` (the tool reads `[defaults]`, `[identity]`, and `[kinematics.*]`).

### `[identity]` *(optional)*

```toml
[identity]
node_id = 0x42
#   7-bit logical MotorAddress / NodeId (0x08..0x77) persisted to NVM.
#   Can also be supplied under [defaults].node_id.
```

---

### `[kinematics]`

Configures runtime actuator topology, sensor-to-motor routing, and transmission parameters.

```toml
[kinematics]
topology = "two_2dof"
#   "four_1dof" (0), "two_2dof" (1), "one_1dof_one_2dof" (2),
#   "two_1dof_one_2dof" (3), or "custom" (0xFF).
sensor_map = [1, 0, 3, 2]
#   4-element array mapping motor channel [0..3] to sensor channel [0..3].
transmission = "coupled_differential"
#   Active DynamicTransmission variant:
#   "linear_gear" (0), "coupled_differential" (1), "lut" (2),
#   "four_bar_linkage" (3), or "eccentric_cam" (4).
gear_ratio = 2.5
#   LinearGear reduction ratio (motor_rad / joint_rad).
backlash_delta = 0.01
#   LinearGear backlash deadband width in radians.
omega_thresh = 0.2
#   LinearGear velocity threshold for directional backlash switching (rad/s).
torsional_stiffness = 120.0
#   LinearGear torsional compliance stiffness (Nm/rad).
joint_offset_alpha = 0.25
#   Zero-position offset for joint axis alpha (radians).
joint_offset_beta = 0.0
#   Zero-position offset for joint axis beta (radians).
outer_downsample = 4
#   Outer joint PID divider relative to inner FOC rate (1..255).
```

---

### `[kinematics.differential]`

2x2 forward transformation matrix $T$ for `transmission = "coupled_differential"`. Accepts either a 2x2 nested array `matrix` or individual `t11`, `t12`, `t21`, `t22` keys.

```toml
[kinematics.differential]
matrix = [
  [0.5,  0.5],
  [0.5, -0.5],
]
#   Or equivalently:
# t11 = 0.5
# t12 = 0.5
# t21 = 0.5
# t22 = -0.5
```

---

### `[kinematics.linkage]`

Link lengths for `transmission = "four_bar_linkage"` or `"eccentric_cam"`.

```toml
[kinematics.linkage]
l1 = 40.0
#   Ground frame link length L1.
l2 = 20.0
#   Input crank link length L2.
l3 = 45.0
#   Coupler link length L3.
l4 = 25.0
#   Output rocker link length L4.
```

---

### `[kinematics.lut]`

Piecewise-linear lookup table (up to 16 points) for `transmission = "lut"`.

```toml
[kinematics.lut]
mot_min = -3.14159
#   Minimum motor input angle in radians.
mot_max = 3.14159
#   Maximum motor input angle in radians.
table = [-1.0, -0.8, -0.5, 0.0, 0.5, 0.8, 1.0]
#   Uniformly spaced joint output samples across [mot_min, mot_max].
```

---

### `[kinematics.outer_pid_alpha]` and `[kinematics.outer_pid_beta]`

Outer-loop joint position PID gains for 1-DOF (`alpha`) and 2-DOF (`alpha`, `beta`) actuators.

```toml
[kinematics.outer_pid_alpha]
p = 20.0
i = 0.0
d = 0.1
limit = 30.0
ramp = 100.0

[kinematics.outer_pid_beta]
p = 20.0
i = 0.0
d = 0.1
limit = 30.0
ramp = 100.0
```

---

## Commissioning Example

```bash
# 1. Discover servos on the I3C bus
python3 test-scripts/configure_fishyservo_i3c.py --scan

# 2. Apply a TOML commissioning profile and persist to Bank 2 Sectors 14 & 15
python3 test-scripts/configure_fishyservo_i3c.py --addr 0x40 --config my_joint.toml --save

# 3. Inspect active configuration and live telemetry
python3 test-scripts/configure_fishyservo_i3c.py --addr 0x40 --show
```
