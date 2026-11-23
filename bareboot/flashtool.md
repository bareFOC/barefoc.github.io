---
layout: default
title: "Host Flashtool Reference"
nav_order: 5
parent: "BareBoot"
permalink: /bareboot/flashtool/
description: "Command-line flashing reference for bareboot-cli and multidrop_flashtool.py across I2C, I3C, UART, and USB"
---

# Host Flashtool Reference

BareBoot provides command-line utilities for automated testing, CI/CD pipelines, and multi-node field deployments.

---

## 1. `bareboot-cli` Reference

The Rust-native `bareboot-cli` executable communicates directly with BareBoot and Katapult bootloaders across serial, I2C, I3C, and CAN interfaces.

### Universal Handsfree Option (`-r`, `--request-bootloader`)
Passing `--request-bootloader` instructs `bareboot-cli` to issue the transport-specific handsfree entry command before initiating Katapult handshake frames:

```bash
# 1. USB CDC (Issues 1200-baud touch reset)
bareboot-cli --firmware app.bin --serial /dev/ttyACM0 --request-bootloader

# 2. UART Serial (Emits in-band break and sync token @ 250,000 baud)
bareboot-cli --firmware app.bin --serial /dev/ttyUSB0 --baud 250000 --request-bootloader

# 3. I2C Target (Emits 0xF0 strobe to address 0x40 on bus /dev/i2c-1)
bareboot-cli --firmware app.bin --i2c /dev/i2c-1 --address 0x40 --request-bootloader

# 4. MIPI I3C (Emits CCC RSTACT defining byte 0x02)
bareboot-cli --firmware app.bin --i3c /dev/i3c-0 --address 0x40 --request-bootloader

# 5. SocketCAN (Transmits admin reboot frame to node UUID)
bareboot-cli --firmware app.bin --can can0 --node-id 0x100 --uuid 123456789abc --request-bootloader
```

---

## 2. Multi-Drop Rollout Orchestration (`multidrop_flashtool.py`)

For multi-actuator robotics networks sharing a common multi-drop I2C or I3C bus, `multidrop_flashtool.py` automates sequential firmware rollouts using declarative YAML manifests:

```yaml
# configs/robot_hand.yaml
bus:
  adapter: ch341 # or linux_i2c
  device: /dev/i2c-1
  speed: 400000

nodes:
  - name: thumb_flex
    address: 0x40
    firmware: builds/thumb.bin
  - name: index_flex
    address: 0x41
    firmware: builds/finger.bin
  - name: middle_flex
    address: 0x42
    firmware: builds/finger.bin
```

### Execution Commands

```bash
# Execute pre-flight bus inventory and sequential multi-node update
python3 host/scripts/multidrop_flashtool.py configs/robot_hand.yaml

# Update a single targeted node within the topology
python3 host/scripts/multidrop_flashtool.py configs/robot_hand.yaml --node index_flex
```

### Rollout Safety Guarantees:
- **Pre-Flight Inventory**: Scans and verifies connectivity to all nodes before rebooting any node. If one node fails discovery, the update halts immediately.
- **Sequential Flashing**: Updates one actuator at a time to prevent bus contention or aggregate power surges.
- **Readback Verification**: Re-reads programmed flash blocks (`CMD_REQUEST_BLOCK = 0x14`) and compares against host SHA256 hashes before issuing `CMD_COMPLETE`.
