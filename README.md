# barefoc.github.io

Documentation website for the [BareFOC](https://github.com/bareFOC/bareFOC) project — pure-Rust, `#![no_std]` Field Oriented Control (FOC) library for embedded systems.

- **Website**: [https://barefoc.github.io](https://barefoc.github.io)
- **Crate**: [barefoc on crates.io](https://crates.io/crates/barefoc)
- **Repository**: [bareFOC/bareFOC](https://github.com/bareFOC/bareFOC)
- **Upstream Reference**: [SimpleFOC](https://simplefoc.com)

## Local Development

The site is built with Jekyll using the GitHub Pages workflow.

```bash
bundle install
bundle exec jekyll serve
```

## Structure

- `index.md`: Documentation homepage and architectural overview.
- `docs/getting_started/`: Quick start guide, Cargo setup, feature flags, and minimal example.
- `docs/theory/`: Mathematical foundations, Clarke/Park transforms, SVPWM, and control loops.
- `docs/actuators/`: BLDC, stepper, hybrid stepper, DC motor control, and kinematics.
- `docs/sensors/`: Encoders, magnetic sensors, Hall sensors, and the `Sensor` trait.
- `docs/drivers/`: 3-PWM, complementary 6-PWM, and smart gate driver ICs.
- `docs/current_sensing/`: Inline, low-side, and current mirror sensing.
- `docs/communication/`: Commander ASCII parser, Fibre RPC, Multibus register table, and RTT.
- `docs/simulation/`: Software-in-the-Loop (`barefoc-sim`) and live GUI tuning.
- `docs/platforms/`: Hardware abstraction bridges for STM32, ESP32, and CH32.
