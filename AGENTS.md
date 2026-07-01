# AGENTS

## Purpose

This repository is the mono-repo for all peripheral firmware in the Greenhouse platform.

Peripherals are Peripheral Units: small microcontroller boards (Arduino Nano / Pro class, occasionally ESP32 for dual-I2C sensors) that are hard-coded for a specific sensor or actuator and attach to an Edge Unit slot over the shared I2C bus.

This is not the Edge Unit firmware repository, and not the Main Unit application repository.

## First Action

Before taking any other action in this repository, agents must read the central documentation entry point:

- https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md

Use the dedicated Greenhouse Documentation repository for durable project documentation, canonical context, the device model, the I2C slot / canonicalization contract, MQTT contracts, ADRs, and skill guidance.

## Cross-Agent Bridges

This file is the canonical, neutral, cross-agent policy for this repository. All coding agents share it:

| Tool | Entry file |
|---|---|
| OpenAI Codex | `AGENTS.md` (this file) |
| Claude / Claude Code | `CLAUDE.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |

`CLAUDE.md` and `.github/copilot-instructions.md` defer to this file; they add tool-specific operational guidance only and never override it. Local skill packs under `docs/skills/` are tool-neutral and apply equally to all agents. When cross-agent policy changes, update this file and keep the bridges in sync rather than duplicating policy per tool.

## Scope Boundaries

- Keep work focused on Peripheral Unit firmware concerns.
- A peripheral is hard-coded for a single piece of hardware. Keep it small, single-purpose, and deterministic.
- Do not introduce Edge Unit responsibilities into a peripheral: no peripheral discovery/registry, no WiFi, no MQTT, no BLE provisioning, no Main Unit UI, and no cloud-first assumptions. Those belong to the Edge Unit and Main Unit respectively.
- The Edge Unit is the I2C master; a peripheral is an I2C slave that exposes a canonical interface and nothing more.
- Keep actuator safety behavior local to the peripheral.

## Instruction Precedence

Use this precedence order when instructions overlap:

1. AGENTS.md (this file)
2. .github/copilot-instructions.md
3. Greenhouse Documentation repository instructions and docs
4. Local repository docs under docs/ (supplemental only)

If guidance conflicts, follow the highest-precedence source.

## Local Docs And Skill Pattern

- Treat all local docs in this repository under docs/ as supplemental repository guidance.
- Do not let local docs override canonical policy, the device model, the I2C/canonicalization contract, or terminology from the Greenhouse Documentation repository.
- If a documentation, knowledge or skill gap is identified, don't make things up, instead bring it to the user's attention to be addressed properly.
- Local documentation authored in this repository would typically be in the form of ADRs located under docs/adr/.
- When adding or updating a local ADR:
	1. Read the central documentation entry point first: https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md
	2. Align terms and assumptions with canonical docs before writing local ADR content.
	3. Keep local ADRs focused on repository-specific decisions that do not belong in shared canonical docs.
	4. If canonical and local guidance differ, follow canonical guidance and update or scope the ADR accordingly.

## Coding Standards

- Use object-oriented design with clear module boundaries, sized to the constraints of small MCUs.
- Program to small interfaces and separate the three peripheral concerns: hardware driver, I2C-slave protocol, and canonicalization.
- Keep declarations in headers and implementations in source files.
- Language is pragmatic per layer (matching the platform convention):
	- Arduino C++ libraries: `.h` declarations with `.cpp` implementations.
	- The `.ino` sketch is a thin entry point only (`setup()` / `loop()` wiring), not a home for logic.
	- C drivers where a C-style HAL is clearer: `.h` declarations with `.c` implementations.
- Isolate hardware access (ADC, GPIO, sensor/actuator transport) behind a driver interface.
- Prefer composition over inheritance; avoid abstraction the part does not need.
- Keep failure paths explicit with stable, canonical status codes.

## Firmware Runtime Rules

- Keep `loop()` non-blocking. Use `millis()`-based scheduling; do not use `delay()` for cadence.
- Keep I2C slave callbacks (`onReceive` / `onRequest`) minimal: copy data, set a flag, and process in `loop()`. Never block, allocate, or do slow IO inside a callback.
- Avoid dynamic allocation and the Arduino `String` class on RAM-constrained boards; use fixed-size buffers and stable, statically-sized message structures.
- Emit only canonicalized values and status across the I2C boundary; never leak raw ADC counts or sensor-specific encodings.
- Maintain actuator fail-safe defaults at power-on, on loss of Edge Unit communication, and on watchdog reset.
- Use the hardware watchdog so a hung peripheral recovers without taking down its slot.
- Preserve the canonical I2C address-family conventions and the slot/canonicalization contract from the Greenhouse Documentation repository.

## Quality Gates

Before finalizing changes, verify:

1. The peripheral remains single-purpose and hard-coded for its one piece of hardware (no creeping Edge Unit responsibilities).
2. Hardware driver, I2C-slave protocol, and canonicalization are separated and individually testable.
3. `loop()` and all I2C callbacks are non-blocking and bounded.
4. RAM and flash usage fit the target board with margin; no `String`/heap growth in steady state.
5. Actuator safe-state defaults hold across power-on, comms loss, and watchdog reset.
6. The exposed I2C interface and canonical values match the device model and contracts in the Greenhouse Documentation repository.
7. Host-side logic (canonicalization, status mapping) has unit tests where practical.
