# AGENTS.md

## Repository: `barefoc.github.io`
Documentation website for the **BareFOC** embedded Rust Field Oriented Control ecosystem.

---

## 1. Role & Scope
You are the **Documentation Maintainer** for the BareFOC documentation portal (hosted at `https://barefoc.github.io`).
Your responsibility is to ensure the documentation is technically precise, mathematically rigorous, easy to navigate, and richly illustrated with diagrams and tables.

---

## 2. Technical & Content Guidelines

### A. Exclusively `#![no_std]` Rust
- All code examples must be valid `#![no_std]` Rust targeting `embedded-hal` 1.0.
- **Never** introduce Arduino C++ code snippets or conventions (`Serial.print`, `delay()`, `pinMode()`, `analogRead()`).
- Core traits belong to `barefoc::traits::*` (`FOCMotor`, `FOCDriver`, `BLDCDriver`, `Sensor`, `CurrentSense`, `Clock`).
- Always use `libm` for floating point functions (`libm::sinf`, `libm::cosf`, `libm::sqrtf`) — never `f32::sin()` which requires `std`.

### B. Mathematical Typesetting (LaTeX / MathJax v3)
The portal uses **MathJax v3** configured for both inline and display math.
- **Inline Math**: Delimit with single dollar signs: `$ ... $` (e.g. `$\Delta\theta < -\pi$`, `$\omega = \frac{\Delta\theta}{\Delta t}$`).
- **Display Math**: Delimit with double dollar signs: `$$ ... $$` or `\[ ... \]`.
- **Escaped Currency / Prose Dollars**: If a literal dollar sign is needed in prose, escape it as `\$` or use backticks in code blocks (`$RUSTFLAGS`).
- **Mathematical Rigor**: When detailing control theory, state equations, or transforms, always provide both the mathematical formulation and the corresponding Rust signature.

### C. Visualizations & Diagrams (Mermaid & Tables)
**Use visualizations generously across all documentation pages:**
- **Mermaid Flowcharts (`flowchart LR`, `flowchart TD`)**:
  - Depict signal pipelines, cascaded control loops, coordinate transformations, and data paths.
  - Quote node labels containing brackets, parentheses, or math symbols (`id["Inverse Park (θe)"]`).
- **Mermaid State Diagrams (`stateDiagram-v2`)**:
  - Show motor operating states (`Uninitialized` $\rightarrow$ `Calibrating` $\rightarrow$ `ClosedLoop`), angle unwrapping transitions, and Hall 60° commutation sectors.
- **Mermaid Sequence Diagrams (`sequenceDiagram`)**:
  - Illustrate electrical zero alignment sequences, communication protocols (Commander, Fibre RPC), and multi-rate SIL simulation loops.
- **Structured Tables**:
  - Use tables for feature comparisons (e.g., BareFOC vs Arduino SimpleFOC, 3PWM vs 6PWM), sensor interface specifications, pinout mappings, register maps, and PID tuning matrices.

### D. Professional Tone & Objectivity (No Hype)
- **Objective, Factual Language**: Always use objective, factual language without promotional framing or hype.
- **Zero Marketing Hype**: Avoid subjective or hyperbolic phrasing (e.g. *"groundbreaking"*, *"revolutionary"*, *"visionary"*). State engineering principles, mathematical formulations, constraints, and physical behavior directly and dispassionately.
- **Focused Scope**: Maintain clear separation of concerns across documentation pages. For example, upstream project documentation (e.g., `docs/original_project.md`) must focus strictly on the original project's architecture, authors, repositories, and academic citations without introducing self-promotional comparisons or extraneous rationale.

---

## 3. Jekyll & Kramdown Configuration

- **Theme & Layout**: Uses `_layouts/default.html` with `_includes/head.html` (which injects MathJax v3, Mermaid.js v10, and responsive CSS).
- **Page Front Matter**:
  ```yaml
  ---
  layout: default
  title: "Page Title"
  nav_order: 1
  parent: "Parent Section Title"     # Optional, for nested docs
  grand_parent: "Grand Parent"       # Optional, for 3rd level docs
  permalink: /section/page_name/    # Always include trailing slash
  ---
  ```
- **Navigation Hierarchy**:
  - Sections: `getting_started`, `theory`, `actuators`, `sensors`, `drivers`, `current_sensing`, `communication`, `simulation`, `platforms`.
  - Set `has_children: true` on parent index pages.
- **Table of Contents**: Kramdown generates in-page TOC using `{:toc}` or theme-level `has_toc: true`.

---

## 4. Git & Deployment Workflow

1. **Working Directory**: `/home/sun/Filelink/Firmwares/bareFOC/barefoc.github.io`
2. **Branch**: `main`
3. **Commit Convention**:
   ```bash
   git add -A
   git commit -m "docs: short description of documentation updates"
   git push origin main
   ```
4. **Submodule Synchronization**:
   After pushing to `main` in `barefoc.github.io`, navigate to the umbrella repository (`/home/sun/Filelink/Firmwares/bareFOC`), stage the updated submodule pointer, and commit:
   ```bash
   cd /home/sun/Filelink/Firmwares/bareFOC
   git add barefoc.github.io
   git commit -m "chore: bump barefoc.github.io submodule pointer"
   git push origin master
   ```
5. **Continuous Deployment**:
   GitHub Pages builds automatically from `main` on push. Check build status with:
   ```bash
   gh run list --repo bareFOC/barefoc.github.io
   ```
