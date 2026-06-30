# CLAUDE

This file is the Claude / Claude Code entry point for the Greenhouse Peripherals firmware repository.

## First Action

Before taking any other action in this repository, read the central documentation entry point:

- https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md

The dedicated Greenhouse Documentation repository is canonical for durable project documentation, platform context, the device model, the I2C slot / canonicalization contract, MQTT contracts, ADRs, and skill guidance. This firmware repository is a consumer of that documentation, not a replacement for it. Treat the Greenhouse Documentation repository as authoritative whenever guidance overlaps.

## Canonical Policy

@AGENTS.md

AGENTS.md is the canonical, cross-agent policy for this repository.
All scope boundaries, instruction precedence, coding standards, firmware runtime rules, and quality gates in AGENTS.md apply fully to Claude and Claude Code.

## What This Repository Is

This repository is the mono-repo for peripheral firmware: the small sensor and actuator MPUs (Arduino Nano / Pro class, occasionally ESP32 for dual-I2C sensors) that attach to an Edge Unit slot over I2C.

It is not the Edge Unit firmware repository and not the Main Unit application repository.

Claude Code may write and edit peripheral firmware code, tests, and local supplemental docs in this repository.
Claude Code must not introduce Edge Unit responsibilities (peripheral discovery, WiFi, MQTT, BLE provisioning), Main Unit UI, or cloud-first assumptions into peripheral tasks.

## Keep Peripherals Small

A peripheral is hard-coded for a single piece of hardware. Favor the simplest design that meets the contract. Do not add discovery, registries, dynamic configuration, or speculative abstraction — that complexity lives on the Edge Unit, not the part.

## Incremental Loading and Partial Disclosure

**Do not read all documentation or all skill packs up front.**

This is the required pattern - load only what the task needs:

1. Always read first:
   - The Greenhouse Documentation `README.md` (see First Action) - it routes to canonical context, the device model, the I2C/canonicalization contract, and the relevant specs.

2. From the documentation repository, load only the canonical docs that match the current task (for example: the device model and the slot/canonicalization contract for a new sensor or actuator). Do not load unrelated specs or orientation docs.

3. In this repository, load only the local skill packs in `docs/skills/` that match the task (see Local Skill Packs below). Do not load all skill packs at once.

4. Load local ADRs under `docs/adr/` only for the area you are touching.

This pattern preserves context and prevents conflicting guidance from non-applicable docs and skills.

## Instruction Precedence

When instructions overlap, follow this order:

1. `AGENTS.md` (canonical cross-agent policy)
2. `CLAUDE.md` (this file - Claude-specific operational guidance; never overrides AGENTS.md)
3. Greenhouse Documentation repository instructions and docs (canonical project documentation)
4. `docs/skills/*.md` (local skill packs - supplemental only)
5. `docs/adr/*.md` (local ADRs - supplemental, repository-specific decisions only)

If guidance conflicts, follow the highest-precedence source. Local docs and skill packs must never override canonical policy, the device model, contracts, or terminology from the Greenhouse Documentation repository.

## Local Skill Packs

Local, implementation-focused skill packs for this repository live under `docs/skills/`.

These are supplemental to the canonical skills in the Greenhouse Documentation repository. Review the relevant canonical skill guidance first, then apply these repository-local packs for peripheral / Arduino specifics.

Each pack is written at the platform-and-pattern level so it applies across peripheral types (any sensor or actuator MPU), not to a single device.

Read `docs/skills/README.md` to select the right pack for the task.
Load only the matching pack, not all of them.

If a documentation, knowledge, or skill gap is identified, do not make things up - bring it to the user's attention to be addressed properly, per AGENTS.md.

## Local ADRs

Repository-specific architectural decisions live under `docs/adr/`.

Read `docs/adr/README.md` first when present - it is the ADR digest and loading rules. Use it to load only the ADRs relevant to the area you are touching, not all of them.

When adding or updating a local ADR, follow the ADR workflow in AGENTS.md: read the central documentation entry point first, align terms and assumptions with canonical docs, keep ADRs focused on repository-specific decisions, and defer to canonical guidance on conflict.

## Repository Tool Bridges

| Tool | File |
|---|---|
| Claude / Claude Code | `CLAUDE.md` (this file) |
| OpenAI Codex | `AGENTS.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |

All bridges point to `AGENTS.md` as the canonical source of policy, and to the Greenhouse Documentation repository as the canonical source of project documentation.
