---
layout: default
title: "Kinematics & Transmissions"
nav_order: 5
parent: "Actuators & Motors"
permalink: /actuators/kinematics/
---

# Kinematics & Transmissions (`barefoc::kinematics`)

The `barefoc::kinematics` module provides `#![no_std]` coordinate transformations, transmission models, multi-axis coupled joint decoupling, cascaded dual-loop controllers, and on-device auto-calibration routines for robotic mechanisms.

---

## Architecture Overview

```mermaid
graph TD
    subgraph TransmissionLayer ["Kinematic Transmission Models"]
        Trait["Transmission&lt;M, J&gt; Trait<br/>forward() • inverse() • jacobian() • reflect_effort()"]
        Linear["LinearGear<br/>Backlash Deadband • Compliance"]
        Diff["CoupledDifferential<br/>Symmetric Wrists • 2x2 Coupled Matrix"]
        Linkage["AnalyticalLinkage<br/>FourBarLinkage (Freudenstein) • EccentricCam"]
        LUT["LutTransmission&lt;N&gt;<br/>O(1) Linear Interpolation • Empirical Profiles"]
        Trait --> Linear
        Trait --> Diff
        Trait --> Linkage
        Trait --> LUT
    end

    subgraph ActuatorLayer ["Cascaded Dual-Loop Actuators"]
        Act1["Actuator1DOF<br/>Inner FOC (10-25 kHz) + Outer Joint PID (500 Hz)<br/>Jacobian Gain Scheduling: v_cmd = v_pid / J(θ)"]
        Act2["Actuator2DOF<br/>Coupled 2-Motor + Differential Decoupling + Sensor2D"]
    end

    subgraph CalibrationLayer ["On-Device Auto-Calibration"]
        Auto["AutonomousCalibrator<br/>Stall Detection • Limit Discovery • LUT Generation"]
        Inter["InteractiveCalibrator<br/>Step-by-Step State Machine"]
        Flash["FlashCalibrationData<br/>64-Byte Quadword-Aligned Flash Header • CRC32"]
    end

    TransmissionLayer --> ActuatorLayer
    ActuatorLayer --> CalibrationLayer
```

---

## 1. The `Transmission<M, J>` Trait

All transmission models implement the generic `Transmission<const MOTORS: usize, const JOINTS: usize>` trait:

```rust
pub trait Transmission<const MOTORS: usize, const JOINTS: usize> {
    /// Forward kinematics: Maps motor rotor angles (rad) to joint output angles (rad)
    fn forward(&self, mot_angles: &[f32; MOTORS]) -> [f32; JOINTS];

    /// Inverse kinematics: Maps joint output angles (rad) to motor rotor angles (rad)
    fn inverse(&self, joint_angles: &[f32; JOINTS]) -> [f32; MOTORS];

    /// Instantaneous Jacobian matrix J: partial derivatives ∂θ_joint[j] / ∂θ_mot[m]
    fn jacobian(&self, mot_angles: &[f32; MOTORS]) -> [[f32; MOTORS]; JOINTS];

    /// Forward velocity projection: v_joint = J * v_mot
    fn forward_velocity(
        &self,
        mot_angles: &[f32; MOTORS],
        mot_velocities: &[f32; MOTORS],
    ) -> [f32; JOINTS];

    /// Effort reflection: projects joint torques to motor torques: τ_mot = J^T * τ_joint
    fn reflect_effort(
        &self,
        mot_angles: &[f32; MOTORS],
        joint_torques: &[f32; JOINTS],
    ) -> [f32; MOTORS];
}
```

### Mathematical Formulation:

- **Position Forward Kinematics**:
  $$\boldsymbol{\theta}_j = f(\boldsymbol{\theta}_m)$$

- **Instantaneous Jacobian Matrix ($\mathbf{J}$)**:
  $$J_{j,m} = \frac{\partial \theta_{j}}{\partial \theta_{m}}, \quad \mathbf{J} \in \mathbb{R}^{J \times M}$$

- **Velocity Mapping**:
  $$\mathbf{v}_j = \mathbf{J}(\boldsymbol{\theta}_m) \cdot \mathbf{v}_m$$

- **Effort Reflection (Virtual Work Duality)**:
  $$\boldsymbol{\tau}_m = \mathbf{J}^T(\boldsymbol{\theta}_m) \cdot \boldsymbol{\tau}_j$$

---

## 2. Transmission Models

| Transmission Type | Motors $\to$ Joints | Model Class | Characteristics |
|---|---|---|---|
| **`LinearGear`** | $1 \to 1$ | Linear / Direct | Constant reduction ratio $R$, smooth backlash deadband, torsional elasticity |
| **`CoupledDifferential`** | $2 \to 2$ | Coupled Linear | 2-DOF differential wrists and gimbals via 2x2 coupling matrices |
| **`FourBarLinkage`** | $1 \to 1$ | Non-linear Analytical | Exact closed-form Freudenstein equation, open/crossed branch selection |
| **`EccentricCam`** | $1 \to 1$ | Non-linear Analytical | Rocker follower angle derived from eccentric crank geometry |
| **`LutTransmission<N>`** | $1 \to 1$ | Non-linear Empirical | $\mathcal{O}(1)$ piecewise linear interpolation over $N$ uniform grid points |
| **`DynamicTransmission<N>`** | $1 \to 1$ / $2 \to 2$ | Runtime Dynamic | Flat enum container wrapping all models for zero-heap runtime reconfiguration |

---

### A. Linear Gear Reduction (`LinearGear`)

Models planetary gearboxes, harmonic drives, belt pulleys, and lead screws. Includes active compensation for **mechanical backlash** and **structural torsional compliance**.

```rust,no_run
use barefoc::kinematics::transmission::LinearGear;

let gearbox = LinearGear::new(50.0)             // 50:1 reduction ratio
    .with_inverted(false)                       // Preserve direction
    .with_zero_offset(0.0)                      // Rotor zero reference
    .with_backlash(0.002, 0.05)                 // δ = 2 mrad deadband, ω_thresh = 0.05 rad/s
    .with_torsional_stiffness(1500.0);          // K_gear = 1500 Nm/rad stiffness
```

#### Backlash & Compliance Formulation:

To prevent discontinuous chatter at velocity zero-crossings, BareFOC models backlash hysteresis using a smooth hyperbolic tangent transition:

$$\Delta\theta_{\text{backlash}} = \delta \cdot \tanh\left(\frac{\omega_m}{\omega_{\text{thresh}}}\right)$$

Torsional gear flex under load is compensated using linear elastic feedforward:

$$\Delta\theta_{\text{flex}} = \frac{\tau_j}{K_{\text{gear}}}$$

The compensated joint output angle is evaluated as:

$$\theta_j = \frac{\theta_m - \theta_0 - \Delta\theta_{\text{backlash}}}{R} - \Delta\theta_{\text{flex}}$$

---

### B. Coupled Differential Transmission (`CoupledDifferential`)

Coordinates 2-motor to 2-joint coupled mechanisms such as robotic pitch/roll differential wrists and spherical camera gimbals.

```mermaid
flowchart LR
    subgraph Motors ["Actuator Inputs"]
        M1["Motor 1 (θm1, τm1)"]
        M2["Motor 2 (θm2, τm2)"]
    end

    subgraph Differential ["2x2 Coupled Matrix T"]
        Coupling["α = (θm1 + θm2) / (2R)<br/>β = (θm1 - θm2) / (2R)"]
    end

    subgraph Joints ["Joint Outputs"]
        J1["Pitch Joint (α, τα)"]
        J2["Roll Joint (β, τβ)"]
    end

    M1 --> Coupling
    M2 --> Coupling
    Coupling --> J1
    Coupling --> J2
```

```rust,no_run
use barefoc::kinematics::transmission::CoupledDifferential;

// Symmetric differential wrist with 10:1 bevel gear reduction
let diff = CoupledDifferential::symmetric(10.0);

// Forward kinematics: [θm1, θm2] -> [Pitch (α), Roll (β)]
let joint_angles = diff.forward_2d([motor1_pos, motor2_pos]);

// Reflect joint torques to motor torques: [τα, τβ] -> [τm1, τm2]
let motor_torques = diff.reflect_torques_2d([pitch_torque, roll_torque]);
```

---

### C. Analytical Linkages (`FourBarLinkage` & `EccentricCam`)

For mechanical linkages with closed-form geometric solutions, BareFOC provides exact analytical kinematics and exact continuous Jacobian derivatives:

#### Four-Bar Linkage (`FourBarLinkage`):
Solves Freudenstein's equation for crank-rocker and double-rocker mechanisms:

$$K_1 \cos\theta_4 - K_2 \cos\theta_2 + K_3 = \cos(\theta_2 - \theta_4)$$

Where the Freudenstein constants are:

$$K_1 = \frac{L_1}{L_2}, \quad K_2 = \frac{L_1}{L_4}, \quad K_3 = \frac{L_1^2 + L_2^2 - L_3^2 + L_4^2}{2 L_2 L_4}$$

```rust,no_run
use barefoc::kinematics::transmission::{FourBarLinkage, FourBarBranch};

// Construct 4-bar linkage: L1=100mm (ground), L2=30mm (crank), L3=90mm (coupler), L4=50mm (follower)
let linkage = FourBarLinkage::new(0.100, 0.030, 0.090, 0.050, FourBarBranch::Open);

let follower_angle = linkage.forward_1d(crank_angle);
let instantaneous_gear_ratio = linkage.jacobian_1d(crank_angle);
```

---

### D. Look-Up Table Transmissions (`LutTransmission<N>`)

When linkage geometries, non-circular cams, or tendon routings lack convenient closed-form expressions, `LutTransmission<const N: usize>` provides $\mathcal{O}(1)$ table lookups with linear interpolation:

```rust,no_run
use barefoc::kinematics::transmission::LutTransmission;

// 65-point empirical transmission curve over [0, π] radians
const N: usize = 65;
let table: [f32; N] = [ /* calibrated joint angles */ 0.0; N ];

let lut = LutTransmission::new(0.0, core::f32::consts::PI, table);

// O(1) piecewise linear interpolation
let joint_angle = lut.forward_1d(motor_angle);
let jacobian = lut.jacobian_1d(motor_angle);
```

---

### E. Dynamic Transmission Container (`DynamicTransmission<LUT_N>`)

In reconfigurable robotic systems and multi-bus field actuators, mechanism kinematics may need to change dynamically at runtime (for example, switching from a direct gear drive to an analytical linkage or an empirical look-up table following auto-calibration). To avoid heap allocations (`Box<dyn Transmission>`) or virtual table dispatch penalties in `#![no_std]` real-time firmware, BareFOC provides the stack-allocated `DynamicTransmission<const LUT_N: usize>` enum:

```rust
#[derive(Clone, Copy, Debug, PartialEq)]
pub enum DynamicTransmission<const LUT_N: usize> {
    /// Linear gear reduction with backlash and compliance.
    Linear(LinearGear),
    /// Coupled 2-DOF differential transmission (wrist / gimbal).
    Differential(CoupledDifferential),
    /// Non-linear uniform Look-Up Table transmission.
    Lut(LutTransmission<LUT_N>),
    /// Planar four-bar linkage (Freudenstein analytical solution).
    FourBar(FourBarLinkage),
    /// Eccentric cam transmission.
    Cam(EccentricCam),
}
```

`DynamicTransmission` implements both `Transmission<1, 1>` and `Transmission<2, 2>`, delegating forward kinematics, inverse kinematics, and Jacobian matrix evaluations with inline static pattern matching:

```rust,no_run
use barefoc::kinematics::transmission::{
    DynamicTransmission, LinearGear, FourBarLinkage, FourBarBranch, Transmission
};

// Initialize with linear gear
let linear = LinearGear::new(10.0);
let mut dyn_tx = DynamicTransmission::<65>::from(linear);

// Evaluate 1-DOF forward kinematics
let joint_pos: [f32; 1] = <DynamicTransmission<65> as Transmission<1, 1>>::forward(&dyn_tx, &[10.0]);

// Reconfigure dynamically at runtime to an analytical four-bar linkage
let four_bar = FourBarLinkage::new(0.120, 0.040, 0.100, 0.060, FourBarBranch::Open);
dyn_tx = DynamicTransmission::FourBar(four_bar);

// Evaluate with new linkage kinematics without rebooting or heap allocation
let linkage_pos: [f32; 1] = <DynamicTransmission<65> as Transmission<1, 1>>::forward(&dyn_tx, &[0.5]);
```

---

## 3. Cascaded Dual-Loop Actuator Control

High-performance robotic actuators typically feature **dual encoders**:
1. An **on-rotor high-speed sensor** (e.g., 14-bit magnetic SSI) feeding the high-frequency motor FOC commutation loop (10 kHz – 25 kHz).
2. An **on-joint output sensor** (mounted after the gearbox/linkage) tracking the true mechanical load position (200 Hz – 1 kHz) free from transmission backlash and structural compliance.

```mermaid
flowchart TD
    subgraph OuterLoop ["Decimated Outer Position Loop (200 - 1000 Hz)"]
        TargetJoint(["Target Joint Angle (θj_ref)"]) --> PosError((+ -))
        JointSensor[("Joint Output Sensor<br/>output_sensor.get_angle()")] --> PosError
        PosError --> PosPID["Position PID<br/>pid_position"]
        PosPID --> RawJointVel["Raw Joint Velocity Command"]
        
        RotorAngle[("Rotor Shaft Angle<br/>motor.shaft_angle()")] --> JacCalc["Jacobian Evaluator<br/>J(θm)"]
        RawJointVel --> GainSched["Jacobian Gain Scheduling<br/>v_mot = v_joint / J(θm)"]
        JacCalc --> GainSched
        GainSched --> Clamp["Velocity Limiter & Saturation"]
    end

    subgraph InnerLoop ["High-Frequency Inner FOC Loop (10 - 25 kHz)"]
        Clamp --> MotorVelocityTarget(["Motor Velocity Command"])
        MotorVelocityTarget --> FOCEngine["Motor FOC Commutation<br/>motor.loop_foc() + motor.move_motor()"]
        FOCEngine --> InverterBridge["Gate Driver & Coils"]
    end
```

### Jacobian Gain Scheduling

For non-linear linkages (cams, 4-bar linkages, tendon spools), the mechanical leverage $J(\theta) = \frac{d\theta_j}{d\theta_m}$ varies continuously across the workspace. A fixed PID gain would result in sluggish response at low leverage and instability/oscillation at high leverage.

`Actuator1DOF` automatically applies **Jacobian gain scheduling**:

$$v_{\text{mot, cmd}} = \frac{v_{\text{PID}}}{J(\theta_m)}$$

Normalizing the outer position loop gain by the instantaneous transmission Jacobian guarantees consistent closed-loop bandwidth across the entire range of motion.

### Actuator Integration Example:

```rust,no_run
use barefoc::kinematics::actuator::Actuator1DOF;
use barefoc::common::pid::PIDController;

let mut actuator = Actuator1DOF::new(motor, transmission, joint_sensor, pid_position)
    .with_downsample(10); // Run outer position loop once every 10 FOC steps

loop {
    // High-frequency loop tick: automatically executes outer loop every 10 ticks
    actuator.step(target_joint_angle, &clock);
}
```

---

### Multi-Axis Actuator Assembly (`ActuatorAssembly<M, S>`)

For robotic platforms requiring coordinated multi-axis motion, dynamic differential decoupling, and arbitrary sensor-to-motor routing, BareFOC provides `ActuatorAssembly<const M: usize, const S: usize>` (defaults to $M=4$ motor axes and $S=4$ sensor channels).

`ActuatorAssembly` supervises:
- Inner high-frequency motor commutation loops (10 kHz - 25 kHz)
- Decimated outer discrete-time joint position PID loops (e.g., 500 Hz - 1 kHz)
- Runtime-reconfigurable kinematic topologies via `TopologyPreset`
- Flexible sensor pool multiplexing (`sensor_map: [u8; M]`)
- Global bus voltage monitoring (`vbus_voltage`) and auto-calibration sequencing

#### Kinematic Topology Presets (`TopologyPreset`)

Mechanisms can be reconfigured dynamically between standard multi-axis arrangements or custom configurations:

| Topology Preset | Value | Coupling Mask | Description |
|---|:---:|:---:|---|
| **`Four1Dof`** | `0` | `0x00` | 4 independent 1-DOF joint axes. Each motor drives a single dedicated joint. |
| **`Two2Dof`** | `1` | `0x03` | 2 independent 2-DOF coupled differential joint pairs: motors `[0, 1]` drive differential gimbal 0 (Pitch/Roll), and motors `[2, 3]` drive differential gimbal 1 (Pitch/Roll). |
| **`One1DofOne2Dof`** | `2` | `0x04` | 1 independent 1-DOF joint (motor 0) and 1 coupled 2-DOF differential joint pair (motors `[1, 2]`). |
| **`Two1DofOne2Dof`** | `3` | `0x02` | 2 independent 1-DOF joints (motors 0 and 1) and 1 coupled 2-DOF differential joint pair (motors `[2, 3]`). |
| **`Custom`** | `0xFF` | User-defined | Custom topology mapping driven by arbitrary coupling bitmasks. |

#### Dynamic Sensor Channel Routing (`TopologyConfig`)

Joint feedback sensors (encoders, resolvers, optical sensors) often differ in physical bus addresses or connector pinouts from motor inverter channels. The `TopologyConfig` descriptor maps sensor channel indices from the global sensor pool to motor/joint indices:

```rust
#[repr(C)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub struct TopologyConfig {
    pub preset: TopologyPreset,
    pub sensor_map: [u8; 4],     // sensor_map[motor_index] = sensor_pool_channel
    pub coupling_mask: u8,
}
```

This mapping allows any sensor channel in the assembly's sensor pool to feed any motor axis:

```rust,no_run
use barefoc::kinematics::assembly::{ActuatorAssembly, TopologyPreset};

let mut assembly = ActuatorAssembly::<4, 4>::new();

// Set active topology to Two2Dof (2x 2-DOF differential gimbals)
assembly.set_topology(TopologyPreset::Two2Dof);

// Route sensor channel 2 to motor axis 0, and sensor channel 0 to motor axis 1
assembly.set_sensor_mapping(0, 2);
assembly.set_sensor_mapping(1, 0);

// Configure outer position loop setpoints (in radians)
assembly.set_joint_target(0, 1.5708); // Gimbal 0 pitch setpoint
assembly.set_joint_target(1, 0.7854); // Gimbal 0 roll setpoint

// Run synchronous step across all motors, sensors, and kinematics loops
assembly.step_all(&mut motors, &mut sensors, &clock);
```

#### Assembly Telemetry Structures

`ActuatorAssembly` maintains continuous real-time state for each axis:
- `motors: [MotorAxisTelemetry; M]`: Target setpoint, measured current, rotor shaft angle, shaft velocity, assigned sensor channel, and velocity limit.
- `joints: [JointTelemetry; M]`: Target position setpoint, present position, present velocity, gear ratio, backlash amplitude, and outer PID gains ($K_p, K_i, K_d$).
- `calib: CalibTelemetry`: Command ID, state machine phase, completion percentage progress (0..100%), and non-volatile flash status.

---

## 4. On-Device Auto-Calibration

BareFOC includes automated calibration state machines that discover mechanical limits, measure hysteresis, and record non-linear transmission curves directly on embedded hardware.

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> FindMinEndstop : start_calibration()
    FindMinEndstop --> FindMinEndstop : Probe at safe velocity with current limit
    FindMinEndstop --> MinStallDetected : Rotor stall confirmed (N samples)
    MinStallDetected --> FindMaxEndstop : Sweep forward toward hard stop
    FindMaxEndstop --> MaxStallDetected : Rotor stall confirmed (N samples)
    MaxStallDetected --> MonotonicityCheck : Validate discovered travel range
    MonotonicityCheck --> BuildLUT : Uniform N-point discretization
    BuildLUT --> Completed : Write FlashCalibrationData (CRC32 verified)
    Completed --> [*]
```

### Autonomous Limit Discovery (`AutonomousCalibrator`):

```rust,no_run
use barefoc::kinematics::calibration::{AutonomousCalibrator, CalibrationConfig};

let config = CalibrationConfig {
    sweep_velocity: 1.0,           // 1.0 rad/s safe search speed
    probe_limit: 1.5,              // Low torque/voltage limit during stall probe
    stall_vel_threshold: 0.05,     // < 0.05 rad/s considered stalled
    stall_confirm_samples: 50,     // 50 consecutive ticks to confirm endstop
    timeout_us: 10_000_000,        // 10s timeout
};

let mut calibrator = AutonomousCalibrator::<65>::new(config);
let result = calibrator.calibrate(&mut motor, &mut joint_sensor, &clock)?;

// Access generated monotonic LUT transmission
let calibrated_lut = result.lut;
```

---

## 5. Non-Volatile Flash Storage (`FlashCalibrationData`)

Calibration results, kinematic parameters, and multi-axis topology routing are packed into a 64-byte quadword-aligned binary header designed for direct programming into microcontroller non-volatile flash memory sectors:

```rust
#[repr(C, align(64))]
pub struct FlashCalibrationData {
    pub magic: u32,               // Magic signature: 0x4246_4F43 ("BFOC")
    pub version: u16,             // Format version (1)
    pub flags: u16,               // Feature flags
    pub gear_ratio: f32,          // Nominal gear ratio
    pub zero_offset: f32,         // Rotor zero offset (rad)
    pub backlash_delta: f32,      // Measured backlash half-deadband (rad)
    pub torsional_stiffness: f32, // Torsional gear stiffness (Nm/rad)
    pub joint_min: f32,           // Discovered minimum joint angle (rad)
    pub joint_max: f32,           // Discovered maximum joint angle (rad)
    pub lut_mot_min: f32,         // LUT range minimum motor angle (rad)
    pub lut_mot_max: f32,         // LUT range maximum motor angle (rad)
    pub lut_points: u16,          // Number of points in LUT table
    pub topology: TopologyConfig, // Multi-axis topology preset and sensor routing
    pub reserved: [u8; 12],       // Quadword alignment padding
    pub checksum: u32,            // IEEE 802.3 CRC32 checksum over preceding 60 bytes
}
```

### Flash Memory Layout Reference:

| Byte Offset | Field Name | Data Type | Purpose |
|:---:|---|---|---|
| `0x00` – `0x03` | `magic` | `u32` | Magic identification word (`0x4246_4F43` / `"BFOC"`) |
| `0x04` – `0x05` | `version` | `u16` | Schema version identifier |
| `0x06` – `0x07` | `flags` | `u16` | Configuration flags (inverted, backlash active, compliance active) |
| `0x08` – `0x0B` | `gear_ratio` | `f32` | Kinematic reduction ratio |
| `0x0C` – `0x0F` | `zero_offset` | `f32` | Motor electrical/mechanical zero offset (rad) |
| `0x10` – `0x13` | `backlash_delta` | `f32` | Half-width of backlash hysteresis deadband (rad) |
| `0x14` – `0x17` | `torsional_stiffness` | `f32` | Torsional compliance $K_{\text{gear}}$ (Nm/rad) |
| `0x18` – `0x1F` | `joint_min`, `joint_max` | `f32`, `f32` | Calibrated mechanical travel limits (rad) |
| `0x20` – `0x27` | `lut_mot_min`, `lut_mot_max` | `f32`, `f32` | Non-linear LUT input range bounds |
| `0x28` – `0x29` | `lut_points` | `u16` | Number of discretized interpolation samples |
| `0x2A` – `0x2F` | `topology` | `TopologyConfig` | Topology preset (`u8`), sensor routing map (`[u8; 4]`), coupling mask (`u8`) |
| `0x30` – `0x3B` | `reserved` | `[u8; 12]` | Reserved padding for 64-byte alignment |
| `0x3C` – `0x3F` | `checksum` | `u32` | IEEE 802.3 CRC32 data integrity checksum over `0x00..0x3C` |

### CRC32 Verification & Serialization APIs

Data integrity is protected with the standard IEEE 802.3 CRC32 polynomial (`0xEDB8_8320` reversed / `0x04C1_1DB7` forward), computed strictly over the 60 payload bytes (`0x00..=0x3B`).

```rust,no_run
use barefoc::kinematics::calibration::storage::{FlashCalibrationData, save_to_flash, load_from_flash};

// 1. Prepare calibration parameters
let mut data = FlashCalibrationData::default();
data.gear_ratio = 42.0;
data.backlash_delta = 0.003;

// 2. Serialize to a 64-byte aligned block with automatically computed CRC32
let block: [u8; 64] = data.serialize_to_block();
// Alternatively via the module-level helper:
// let block = save_to_flash(&data);

// 3. Write 'block' directly into MCU flash page via embedded-storage / platform driver
// flash_driver.write(FLASH_SECTOR_ADDR, &block)?;

// 4. Restore and verify on boot
let restored = FlashCalibrationData::deserialize_from_block(&block)
    .expect("Flash calibration valid and CRC32 verified");

// 5. Apply restored topology and kinematics directly to the actuator assembly
assembly.load_from_flash(&restored);
```
