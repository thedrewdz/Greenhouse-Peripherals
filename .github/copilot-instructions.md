# Copilot Instructions

## Canonical Agent Policy

Use AGENTS.md at the repository root as the canonical source of long-lived agent instructions.

Before taking any other action in this repository, read:

- https://github.com/thedrewdz/Greenhouse-Documentation/blob/main/README.md

## Repository Focus

This repository is the mono-repo for peripheral (sensor / actuator MPU) firmware in the Greenhouse platform.

A peripheral is hard-coded for a single piece of hardware, runs on small MCUs (Arduino Nano / Pro class, occasionally ESP32 for dual-I2C sensors), and acts as an I2C slave to its hosting Edge Unit.

Do not introduce Edge Unit responsibilities (peripheral discovery, WiFi, MQTT, BLE provisioning), Main Unit UI, or cloud-first assumptions into peripheral tasks.

## Required References

Read and follow:

- AGENTS.md

Use the dedicated Greenhouse Documentation repository for durable project documentation, canonical context, the device model, the I2C / canonicalization contract, MQTT contracts, ADRs, and skill guidance.

For local implementation skill packs, use docs/skills/ after reviewing canonical skills in the main documentation repository.
