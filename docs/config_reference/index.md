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

### `[bootloader]` *(optional)*

Configures bootloader handoff, boot window, and SRAM mailbox synchronization.

```toml
[bootloader]
enabled = true
#   Whether the application links behind a bootloader slot. Default: true.
type = "bareboot"
#   Bootloader type: "bareboot" or "katapult". Default: "bareboot".
transport = "i3c"
#   Bootloader bus transport: "i3c", "i2c", "uart", or "can". Default: "uart".
boot_window_ms = 250
#   Bootloader wait window before jumping to app, in milliseconds. Default: 250.
#baud_rate = 250000
#   UART baud rate when transport = "uart". Default: 250000.
i2c_address = 0x40
#   Static/proposed 7-bit I2C or I3C address. Optional.
mailbox_address = 0x20007FE0
#   SRAM address of the soft-reset / boot-loop mailbox. Default: 0x20007FF8.
mailbox_magic = 0x5984E3FA6CA1589B
#   64-bit magic signature requesting bootloader entry. Default: 0x5984E3FA6CA1589B.
```

---

### `[safety]`

Hardware protection limits enforced by the FOC safety supervisor.

```toml
[safety]
max_voltage_v = 6.0
#   Hard maximum bus/phase voltage clamp in Volts. Required.
max_current_a = 2.0
#   Hard maximum phase/bus current limit in Amperes. Required.
max_velocity_rad_s = 50.0
#   Maximum allowed angular velocity in rad/s. Required.
thermal_cutoff_c = 85.0
#   Over-temperature shutdown threshold in degrees Celsius. Optional.
default_voltage_limit = 5.0
#   Initial runtime voltage limit in Volts. Required.
default_current_limit = 1.0
#   Initial runtime current limit in Amperes. Required.
```

---

### `[defaults]` *(optional — factory fallback)*

Factory defaults used **only** when NVM (Bank 2 Sectors 14 & 15) is blank or invalid. Once commissioned over I3C (`SaveConfig`), persisted NVM values override these defaults on every boot.

```toml
[defaults]
node_id = 64
#   Default 7-bit bus node ID. Default: 1.
pole_pairs = 1
#   Motor electrical pole pairs. Default: 7.
topology = "four_1dof"
#   Multi-axis actuator topology: "four_1dof", "two_2dof",
#   "one_1dof_one_2dof", "two_1dof_one_2dof", or "custom". Default: "four_1dof".
sensor_map = [0, 1, 2, 3]
#   4-element mapping from motor channel index [0..3] to sensor channel index.
#   Default: [0, 1, 2, 3].
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
gear_ratio = 1.0
#   Default mechanical reduction ratio (motor_rad / joint_rad). Default: 1.0.
#soft_limit_min_rad = -3.14159
#   Optional minimum software position limit in radians.
#soft_limit_max_rad = 3.14159
#   Optional maximum software position limit in radians.
```

---

### `[gate_driver]` and `[[gate_driver.channels]]`

Defines the power stage topology, PWM frequency, and per-channel bridge pins.

```toml
[gate_driver]
driver_type = "drv8411a"
#   Gate driver IC ("drv8411a", "drv8311", "drv8316", "3pwm", "6pwm", etc.). Required.
pwm_frequency_hz = 25000
#   PWM carrier frequency in Hz. Required.
pwm_timer = "TIM1+TIM3"
#   MCU timer peripheral(s) driving PWM outputs. Required.
inh_a = "PA15"
inh_b = "PA8"
inh_c = "PA6"
#   Primary phase / high-side PWM pins. Required.
inl_a = "PB5"
inl_b = "PB15"
inl_c = "PA7"
#   Complementary / low-side or enable pins. Optional.
sleep_pin = "PA9"
#   Active-low sleep / driver enable GPIO pin. Optional.
#fault_pin = "PB12"
#   Hardware nFAULT input pin. Optional.

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

### `[sensor]` and `[[sensor.channels]]`

Defines the primary feedback sensor bus and per-channel sensor addresses or mux slots.

```toml
[sensor]
sensor_type = "tmag5273"
#   Position sensor type ("tmag5273", "mt6701", "as5600", "as5048a", "abi", etc.). Required.
bus = "I2C1"
#   MCU bus peripheral ("I2C1", "SPI2", etc.). Required.
#cs_pin = "PB12"
#   SPI chip-select pin (for SPI sensors). Optional.
clk_pin = "PB6"
#   SCL / SCK pin name. Optional.
data_pin = "PB7"
#   SDA / MOSI / MISO pin name. Optional.
cpr = 65536
#   Counts per revolution (e.g., 65536 for 16-bit, 16384 for 14-bit). Required.

[[sensor.channels]]
#   Optional per-channel sensor descriptors for multi-axis boards.
channel = 0
bus = "I2C1"
address = 0x35
mux_channel = 0
```

---

### `[connectors]` *(optional)*

Host-facing communication connector and default bus speed.

```toml
[connectors]
bus_type = "i3c"
#   Host bus type: "i3c", "i2c", "uart", "can", or "spi". Default: "uart".
tx_scl_pin = "PB3"
#   TX / SCL pin name. Optional.
rx_sda_pin = "PB4"
#   RX / SDA pin name. Optional.
default_baud = 1000000
#   Default bus bitrate in Hz (e.g., 1000000 = 1 MHz I3C SDR). Default: 250000.
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
