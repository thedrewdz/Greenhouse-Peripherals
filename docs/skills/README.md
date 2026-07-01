# Local Skill Packs

This folder contains local, implementation-focused skill packs for the Greenhouse Peripherals firmware repository.

These packs are tool-neutral: they work for both OpenAI Codex (via `AGENTS.md` and `.codex` agent profiles) and Claude / Claude Code (via `CLAUDE.md`), and the GitHub Copilot bridge (`.github/copilot-instructions.md`) defers to the same policy. When updating skill guidance, update the shared pack rather than duplicating it per tool, so multi-agent behavior stays consistent.

They are **supplemental**. The canonical skills, context, architecture, device model, and contracts live in the Greenhouse Documentation repository:

- https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md

Review the relevant canonical skill guidance first, then apply these repository-local packs for peripheral and Arduino specifics. Local packs must never override canonical policy, the device model, contracts, or terminology.

## Scope: General, Not Per-Device

Each pack is written at the platform-and-pattern level so it applies across Peripheral Unit types (sensor or actuator), not a single device. A pack describes the reusable rules (structure, I2C-slave protocol discipline, canonicalization, safety, resource budgets); the specific sensor or actuator hardware is supplied per project, not baked into the skill. When you find yourself wanting device-specific guidance, that belongs in the code or an ADR, not in a skill pack.

## Partial Disclosure

Load only the pack that matches the current task. Do not read all packs up front. Each pack states when to use it and when not to.

## Packs

| Pack | Use when |
|---|---|
| `arduino-peripheral-firmware.md` | Writing or refactoring any sensor or actuator peripheral firmware: sketch/library structure, non-blocking scheduling, I2C-slave protocol, canonicalization, actuator safety, watchdog, and fitting RAM/flash budgets. |

## Gaps

If a documentation, knowledge, or skill gap is identified, do not make things up - bring it to the user's attention to be addressed properly, per `AGENTS.md`.
