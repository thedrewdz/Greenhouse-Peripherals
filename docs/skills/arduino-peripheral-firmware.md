# Skill: Arduino Peripheral Firmware

## Purpose

Guide coding agents to implement maintainable, resource-safe firmware for Greenhouse Peripheral Units using the Arduino framework on small MCUs (Arduino Nano / Pro class, occasionally ESP32 for dual-I2C sensors).

A peripheral is an I2C slave to its hosting Edge Unit. It owns one piece of hardware, exposes a canonical interface, and does nothing else. This pack covers how to build that cleanly without over-engineering or exceeding the board's resources.

The I2C address conventions, the slot model, and the canonicalized value/status contract are owned by the Greenhouse Documentation repository (device model and contracts). Read that contract first; this pack covers how to implement against it on an Arduino-class part, not how to define it.

- Canonical entry point: https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md

## Use This Skill When

- Creating a new sensor or actuator peripheral, or refactoring an existing one.
- Implementing the I2C-slave interface a peripheral exposes to its Edge Unit.
- Converting raw hardware readings or commands to/from canonical values and status.
- Implementing actuator safe-state, watchdog, and fault behavior.
- Reviewing peripheral firmware for structure, resource use, and determinism.

## Do Not Use This Skill When

- The task is Edge Unit firmware (peripheral discovery, registry, WiFi, MQTT, BLE provisioning) — that lives in the Edge Unit firmware repository and its skill packs.
- The task is defining the I2C address families, slot model, or canonical value/status contract — those are canonical; raise gaps against the Greenhouse Documentation repository.
- The task is Main Unit UI or any cloud/server concern.

## Keep It Small (Anti-Bloat)

- A peripheral is hard-coded for a single piece of hardware. Choose the simplest design that meets the contract.
- Do not add discovery, registries, plugin systems, dynamic reconfiguration, or speculative abstraction. That complexity belongs on the Edge Unit, not the part.
- Add an abstraction only when it removes real duplication or makes a concern testable — not on principle.
- Prefer a few small, named functions and structs over class hierarchies. Composition over inheritance; no inheritance unless it earns its place.

## Module Structure Rules

- Keep the three peripheral concerns separated, each in its own header/source pair:
  1. **Hardware driver** — talks to the specific sensor or actuator (ADC, GPIO, transport).
  2. **I2C-slave protocol** — handles requests/responses from the Edge Unit.
  3. **Canonicalization** — converts raw readings/commands to/from the canonical value and status contract.
- The `.ino` sketch is a thin entry point only: pin/config constants, object wiring, and `setup()`/`loop()` orchestration. No business logic in the sketch.
- Language is pragmatic per layer: Arduino C++ libraries as `.h`/`.cpp`; a C-style HAL as `.h`/`.c` where that is clearer. Keep declarations in headers and non-trivial implementations in source files.
- Put hardware constants (pins, addresses, thresholds, timings) in one config header so retargeting hardware is a single, reviewable edit.
- Use `#pragma once` or include guards consistently.

## Non-Blocking Runtime Rules

- Keep `loop()` non-blocking. Drive cadence with `millis()` elapsed-time checks, not `delay()`.
- Never block waiting on hardware in the main path; use bounded reads with timeouts and classify timeout separately from invalid data.
- Avoid unbounded loops and recursion. Every loop has a clear bound.
- Keep behavior deterministic: the same inputs produce the same canonical outputs and status.

## I2C-Slave Protocol Rules

- The Edge Unit is the I2C master; the peripheral is the slave. The peripheral never initiates bus transactions to the Edge Unit.
- Use a fixed I2C address drawn from the canonical address family for the peripheral's capability. Validate the configured address against the documented range.
- Keep `Wire.onReceive` / `Wire.onRequest` callbacks minimal and ISR-safe: copy bytes into a buffer, set a flag, and return. Do all parsing, sensing, and actuation in `loop()`.
- Never block, allocate, do slow IO, or call `Serial`/logging inside an I2C callback.
- Use fixed-size, statically allocated message buffers. Bound and validate every inbound payload; reject empty or oversized requests with a stable status code.
- Always have a well-defined response ready for `onRequest` (last good canonical value/status), so a read never stalls the bus.

## Canonicalization Rules

- Cross the I2C boundary only with canonical values and status as defined by the contract. Never leak raw ADC counts, sensor-specific encodings, or device-specific command formats.
- Keep canonicalization in pure functions (raw -> canonical, canonical command -> hardware action) so it can be unit-tested off-device.
- Report identity and health the contract expects (for example capability, firmware version, fault/status code). Surface a per-peripheral fault state rather than failing silently.
- Map all driver errors to stable, canonical status codes; do not invent ad-hoc values.

## Actuator Safety Rules

- Define and apply a safe state at power-on, before the first valid command is received.
- Return to safe state on loss of Edge Unit communication (no valid command within a bounded window), on invalid commands, and on watchdog reset.
- Examples of safe defaults: pumps OFF, heaters OFF, valves CLOSED. Keep safety logic local to the peripheral — it must not depend on the Edge Unit being reachable.
- Enable the hardware watchdog so a hung peripheral resets into its safe state instead of taking down its slot.

## Resource Budget Rules

- Know the target board's limits (for example an ATmega328-class Nano has ~2 KB SRAM, ~32 KB flash) and leave headroom.
- Avoid dynamic allocation and the Arduino `String` class; use `char[]` buffers and fixed structures. No heap growth in steady state.
- Prefer integer and fixed-point math over floating point on AVR-class parts where the canonical format allows.
- Keep RAM stable over long runs — no leaks, no fragmentation, no growing buffers.
- Watch flash and SRAM in the build output as part of every change; treat shrinking margin as a review signal.

## Testing Rules

- Unit-test the pure logic off-device (host build): canonicalization both directions, status/error mapping, command validation, safe-state selection.
- Keep hardware-dependent code thin so most logic is testable without the board.
- Use on-hardware smoke checks for the parts that cannot be mocked: I2C address responds, read returns canonical value, command actuates and reports status, comms-loss drops to safe state, watchdog recovers.

## Multi-Agent Consistency

- This pack is tool-neutral and is the single source of peripheral implementation guidance for both Claude (via `CLAUDE.md`) and Codex (via `AGENTS.md`); Copilot defers to the same policy. Update this shared pack rather than duplicating guidance per tool, so all agents behave consistently.
- This pack is supplemental. On any conflict, follow canonical policy and the device model / contracts in the Greenhouse Documentation repository.

## Validation Checklist

- Peripheral is single-purpose; no Edge Unit responsibilities or speculative abstraction crept in.
- Hardware driver, I2C-slave protocol, and canonicalization are separated and individually testable.
- `loop()` and all I2C callbacks are non-blocking, bounded, and ISR-safe.
- Only canonical values/status cross the I2C boundary; raw encodings stay internal.
- Actuator safe-state holds on power-on, comms loss, invalid command, and watchdog reset.
- RAM and flash fit the target with margin; no `String`/heap growth in steady state.
- Pure logic has off-device unit tests; hardware-only paths have a smoke check.
- Exposed interface, address family, and canonical values match the Greenhouse Documentation contracts.
