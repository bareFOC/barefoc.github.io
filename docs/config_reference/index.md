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

## Compile-Time BSP Codegen (`bareFOC/build.rs`)

When building the `bareFOC` firmware runner, `bareFOC/build.rs` reads the manifest specified by `BAREFOC_BOARD` (defaulting to `examples/microspora/barefoc.toml`) via `bareconfig::bsp::schema::parse_barefoc_toml()` and generates three compile-time artifacts:

1. **`$OUT_DIR/memory.x`**: Linker script with `FLASH`, `RAM`, and NVM sector bounds derived from `[board]` and `[memory]` (using `flash_origin` for standalone builds or `app_origin` when `bootloader = true` or `--features bootloader`).
2. **`$OUT_DIR/bsp_generated.rs`**: Included as `crate::bsp` in `bareFOC/src/main.rs`, containing:
   - Board, memory, bootloader, safety, defaults, gate-driver, communication, and sensor constants (`BOARD_NAME`, `SYSTEM_CLOCK_HZ`, `APP_ORIGIN`, `NVM_SECTOR_ORIGIN`, `POLE_PAIRS`, `PWM_FREQUENCY_HZ`, `COMM_BITRATE`, `SUPPORTED_SENSORS_MASK`, etc.).
   - Typed `embassy_bsp` bindings (`CommIrqs`, `take_comm_peripherals!`, `take_gate_driver_peripherals!`, `take_sensor_peripherals!`, `SharedSpiCsPins`).
   - `create_default_control_table()` initializing the 512-byte control table with factory defaults from `barefoc.toml`.
3. **Capability `--cfg` Flags**: Emitted to `rustc` so `bareFOC/src/runners/` selects generic motor, sensor, and transport runners with **zero per-board `.rs` files**:
   - **MCU**: `mcu_stm32h503cb`, `mcu_stm32g431cb`
   - **Bootloader**: `has_bootloader`
   - **Communication**: `bus_i3c`, `bus_i2c`, `bus_uart`, `bus_can`
   - **Gate Driver**: `drv_drv8411a`, `drv_drv8316`, `drv_drv8311`
   - **Sensor**: `sensor_tmag5273`, `sensor_mt6701`

---

## Board Hardware Sections (`barefoc.toml`)

### `[board]`

Defines the board identity and silicon target.

```toml
[board]
name = "fishyservo-stm32h5"
#   Human-readable board identifier. Required.
description = "FishyServo Quad H-Bridge Multi-Topology Servo (STM32H503KB)"
#   Short description shown in bareConfig and build metadata. Optional.
mcu = "stm32h503kbu6"
#   Target MCU part number (used to auto-derive family, system_clock_hz, and [memory]). Required.
#family = "stm32h5"
#   HAL family identifier (auto-inferred from mcu when omitted).
#system_clock_hz = 250000000
#   Core system clock frequency in Hz (auto-inferred from mcu when omitted).
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

Semantic named communication ports (`[communication.i3c2]`, `[communication.i2c1]`, `[communication.uart1]`, `[communication.fdcan1]`, `[communication.spi1]`, etc.). Both `protocol` and numbered `peripheral` are automatically inferred from the section key prefix (`i3c*`, `i2c*`, `uart*`/`usart*`, `spi*`, `can*`/`fdcan*`) when omitted. Uses protocol-native pin names (`scl_pin`/`sda_pin` for I2C/I3C, `tx_pin`/`rx_pin` for UART/CAN, `sck_pin`/`miso_pin`/`mosi_pin` for SPI) with Klipper-style `!`/`^`/`~` modifiers. Shared identically with `bareboot.toml`.

```toml
[communication.i3c2]
#   Infers protocol = "i3c", peripheral = "I3C2", role = "target".
bootloader = true
#   Allow bootloader access on this port (valid only when role is "target" or "both").
scl_pin = "PB3"
sda_pin = "PB4"
bitrate = 1000000
address = 0x40

[communication.i2c1]
#   Infers protocol = "i2c", peripheral = "I2C1".
role = "master"
scl_pin = "PB6"
sda_pin = "PB7"
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

### `[gate_driver]` and `[gate_driver.<name>]`

Defines the power stage topology (`"h_bridge"` or `"half_bridge"`), optional IC `type` (default `"generic"`), default PWM frequency (`pwm_frequency_hz = 24000`, also runtime-configurable while disarmed via `REG_PWM_FREQUENCY_HZ` at `0x5C`), parallel drive pin arrays, and inline current-sense configuration. PWM timers and channels are automatically assigned at compile time from the MCU pin-AF table (including GPIO `DIR` detection such as `PA15` on FishyServo).

- **Complementary 6-PWM (`half_bridge`) or 2-PWM (`h_bridge`)**: `high_pwm_pin = [...]`, `low_pwm_pin = [...]`
- **3-PWM + GPIO Enable (`half_bridge`)**: `pwm_pin = [...]`, `en_pin = [...]`
- **Sign-Magnitude (`h_bridge`)**: `pwm_pin = [...]`, `dir_pin = [...]`
- **Current Sensing**: `current_sense = "lowside" | "highside" | "inline" | "none"` (defaults to `"lowside"` when `current_adc_pin` is non-empty), `current_adc_pin = [...]`, `csa_gain`, `ripropi_ohms`, `shunt_ohms`.

```toml
[gate_driver]
type = "drv8411a"
topology = "h_bridge"
#pwm_frequency_hz = 24000
high_pwm_pin = ["PA9", "PA8", "PA6", "PA15"]
low_pwm_pin = ["PB0", "PB15", "PA7", "PB5"]
current_sense = "lowside"
current_adc_pin = ["PA5", "PA3", "PA4", "PA2"]
ripropi_ohms = 1000.0
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
