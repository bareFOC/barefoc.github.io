---
layout: default
title: "Contributing & Community"
nav_order: 20
description: "Community, issues, and contributing to BareFOC."
permalink: /contact/
---

# Contributing & Community

BareFOC is an open-source project developed for the embedded Rust and motor control communities.

---

## GitHub Issues & Discussions

- **Repository**: [https://github.com/bareFOC/bareFOC](https://github.com/bareFOC/bareFOC)
- **Bug Reports & Feature Requests**: Submit issues via the [GitHub Issue Tracker](https://github.com/bareFOC/bareFOC/issues).
- **Discussions**: Architectural proposals, questions, and hardware showcase in [GitHub Discussions](https://github.com/bareFOC/bareFOC/discussions).

---

## Upstream SimpleFOC Community

For general motor control theory, hardware shield fabrication questions, and motor tuning discussions, the upstream SimpleFOC community remains an invaluable resource:

- **SimpleFOC Community Forum**: [https://community.simplefoc.com](https://community.simplefoc.com)
- **SimpleFOC Main Site**: [https://simplefoc.com](https://simplefoc.com)

---

## Contribution Guidelines

Contributions are welcome! Before submitting a pull request:
1. Ensure all code complies with `#![no_std]`.
2. Verify that floating-point math uses `libm` and contains zero dynamic memory allocations.
3. Add unit tests running on the host under `cargo test` where applicable.
4. Format your code using `cargo fmt` and verify with `cargo clippy`.
